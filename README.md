# hexo-custom-renderer
custom renderer that  rendered by markdown-it and markdown-it-multimd-table library.

# hexo plugin (scripts/md_renderer.js)
```
'use strict';

hexo.config.markdown = Object.assign({
  render: {},
  plugins: {}
}, hexo.config.markdown);

hexo.config.markdown.render = Object.assign({
  html: true,
  xhtmlOut: false,
  breaks: true,
  linkify: true,
  typographer: true,
  quotes: '“”‘’',
  tab: ''
}, hexo.config.markdown.render);

const renderer = function (data, options) {
	const cfg = this.config.markdown;
	//set markdown-it-multimd-table plugin for markdown-it
	const opt = cfg ? cfg : 'default';
    let parser = (opt === 'default' || opt === 'commonmark' || opt === 'zero' )?
        require('markdown-it')(opt) :
        require('markdown-it')(opt)
            .use(require('markdown-it-multimd-table'), {
              multiline:  true,
              rowspan:    true,
              headerless: false,
              multibody:  true,
              aotolabel:  true,
            })
	//filter -tx- tag
	let mdstr = data.text.replaceAll('-tx-','\n');
			
	//inset toc before content
	let toc = require('markdown-toc');
	let tocmdstr =  "*Summary*\n"+toc(mdstr).content+mdstr
	
	//call cherrio to replace heading tags
	let cheerio = require('cheerio');
	let $ = cheerio.load(parser.render(tocmdstr));
	let headings = $('h1, h2, h3, h4, h5, h6');
    headings.each(function () {
			var $title = $(this);
			var title = $title.text();
			var id = toc.slugify(title, {});
			$title.html( '<span id="' + id + '">' + $title.html() + '</span>' );
	});

   return $.html();
}

//confing custom markdown render for hexo
hexo.extend.renderer.register('md', 'html', renderer, true);
hexo.extend.renderer.register('markdown', 'html', renderer, true);
hexo.extend.renderer.register('mkd', 'html', renderer, true);
hexo.extend.renderer.register('mkdn', 'html', renderer, true);
hexo.extend.renderer.register('mdwn', 'html', renderer, true);
hexo.extend.renderer.register('mdtxt', 'html', renderer, true);
hexo.extend.renderer.register('mdtext', 'html', renderer, true);
```
