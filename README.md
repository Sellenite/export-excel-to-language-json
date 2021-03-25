# export-excel-to-language-json

必须用以下规范编写excel：

| LANGUAGE_KEY | en                | cn        |
| ------------ | ----------------- | ----------|
| key值        |  language en       |language cn|

LANGUAGE_KEY为默认值，如果需要变更的话需要改代码的LANGUAGE_KEY

其他列的第一行作为每个语言的标识符，生成文件会生成类似lang-en.json、lang-cn.json等，支持多个



如果点击下载无反应，可以检查下浏览器右上角是否禁止了多个文件下载，允许后再点击下载即可