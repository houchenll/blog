
# 建站总结

### step 1
以houchen8902.github.io为名，创建新工程，并选择主题后，项目中有两个文件README.md和_config.yml。
_config.ml中只有一句话`theme: jekyll-theme-cayman`，指定主题为cayman。
README.md显示一篇文章。
这时访问https://houchen8902.github.io/，会显示出网页，头部的标题显示工程名，头部的summary显示工程的description。页面内容显示的是README.md中的内容。而整个页面的格式使用的是_config.yml中指定的cayman。

### step 2
在工程根目录添加index.html文件，里面添加一个标题和一个段落。
此时再访问https://houchen8902.github.io/，整个页面就显示index.html的内容。第一，不再显示README.md，第二，不再使用cayman样式。

### step 3
为index.html指定样式。
`<link rel="stylesheet" type="text/css" href="./index.css">`
在根目录创建index.css样式表，并为段落修改样式为蓝色，然后在index.html中引用index.css。
此时再访问页面，显示的段落文字颜色就变成了蓝色。

### step 4
添加子页面跳转
添加子页面child.html，在index.html中添加跳转链接，链接使用相对路径接向子页面，如下。
`<a href="./child.html">静夜思*李白</a>`
重新访问页面，可以看到链接，点击会跳转到child.html页面，跳转后的链接地址为`https://houchen8902.github.io/child.html`。

### step 5
添加子页面绝对路径跳转
在根目录创建pages目录，复制child.html到pages目录。在index.html中添加链接，路径使用绝对地址'/pages/child.html'。
重新访问页面，点击使用绝对路径的链接，可以跳转到子页面，子页面链接地址为`https://houchen8902.github.io/pages/child.html`。

### step 6
添加index.md，修改index.html文件名为hello.html，刷新页面，显示的内容为index.md中的内容。
jkeyll会自动解析index.md中的内容并显示，并应用cayman样式。头部显示与step1相同。
说明显示优先级 index.md > README.md。

### step 7
把hello.html修改为index.html，刷新页面，显示的是index.html中的内容，显示效果与step 5相同。
说明显示优先级 index.html > index.md > hello.html

### step 8
创建目录hello, 将index.html和index.md移入其中，然后刷新显示README.md中的内容，并应用cayman样式。
说明显示优先级README.md > child.html。

### step 9
将README.md移到hello文件中，此时工程根目录只有child.html和note.md，刷新，页面显示404 not found，表明gitpages只能以index.html, index.md, README.md为首页，且优先级顺序为:
index.html > index.md > README.md

### step 10
搭建jekyll站点目录结构
1. 删去除README.md， _config.yml, index.html外的其它文件
2. 按http://jekyllrb.com/docs/structure/提示的结构，创建目录结构
3. 修改/_layouts/default.html，指定title为{{page.title}}, body为{{content}}。
4. 修改/_posts/xxx.md。在文章开头，有一个yaml文件头，用来设置一些元数据。它用三根短划线"---"和标记开始和结束，里面的每一行设置一种元数据。"layout: default"表示该文章的模板使用_layouts目录下的default.html文件。 "title: xxx"表示该文章的标题是xxx，如果不设置这个值，默认使用嵌入文件名的标题。在yaml文件头的后面，就是文章的正式内容，里面可以使用模板变量。{{page.title}}就是文件头中设置的标题，{{page.date}}则是嵌入文件名的日期，也可以在文件头重新定义date变量。 "| date_to_string"表示将page.date变量转换成人类可读的格式。
5. 修改index.html，遍历posts，显示成列表。
刷新显示时，可以看到posts显示在列表中，点击可跳转到相关Post页面，但问题是，设置的cayman主题没有起作用，页面显示是空白，没有主题。

### step 11
修复主题显示
step 10最后显示不了主题，是因为使用layouts中定义的default.html作为主题，所以不指定layout，并且把layout中的default.html修改文件名即可。并且需要把index.html修改为index.md，否则主题还是会显示不出来
修改后的