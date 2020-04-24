# 创世项目文档

### 准备
* [下载](https://desktop.github.com)安装Github Desktop
* [下载](https://www.python.org/downloads)安装Python
* [下载](https://code.visualstudio.com)安装VS Code
* 通过 `pip install --upgrade pip` 更新Python安装工具（Mac OS使用 `pip3`），如果出现*No module named 'pip' error*错误，通过命令 `python -m ensurepip` 修复
* 使用命令 `pip install mkdocs` 安装MKDocs
* 使用命令 `pip install mkdocs-material` 安装mkdocs-material
* 使用命令 `pip install jieba` 安装解霸

#### 添加中文支持
##### 进入python的安装目录修改search_index.py文件
Mac目录为  
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/mkdocs/contrib/search/，修改generate_search_index  
，Windows系统参考相对路径
``` py
    def generate_search_index(self):
        """python to json conversion"""
        page_dicts = {
            'docs': self._entries,
            'config': self.config
        }
        for doc in page_dicts['docs']: 
            tokens = list(set([token.lower() for token in jieba.cut_for_search(doc['title'].replace('\n', ''), True)]))
            if '' in tokens:
                tokens.remove('')
            doc['title_tokens'] = tokens

            tokens = list(set([token.lower() for token in jieba.cut_for_search(doc['text'].replace('\n', ''), True)]))
            if '' in tokens:
                tokens.remove('')
            doc['text_tokens'] = tokens

        data = json.dumps(page_dicts, sort_keys=True, separators=(',', ':'), ensure_ascii=False)

        if self.config['prebuild_index']:
            try:
                script_path = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'prebuild-index.js')
                p = subprocess.Popen(
                    ['node', script_path],
                    stdin=subprocess.PIPE,
                    stdout=subprocess.PIPE,
                    stderr=subprocess.PIPE
                )
                idx, err = p.communicate(data.encode('utf-8'))
                if not err:
                    idx = idx.decode('utf-8') if hasattr(idx, 'decode') else idx
                    page_dicts['index'] = json.loads(idx)
                    data = json.dumps(page_dicts, sort_keys=True, separators=(',', ':'), ensure_ascii=False)
                    log.debug('Pre-built search index created successfully.')
                else:
                    log.warning('Failed to pre-build search index. Error: {}'.format(err))
            except (OSError, IOError, ValueError) as e:
                log.warning('Failed to pre-build search index. Error: {}'.format(e))

        return data
```

##### 修改lunr.js
Mac目录为  
/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/mkdocs/contrib/search/templates/search/  
搜索lunr.Builder.prototype.add替换部分代码

``` js
// 仅替换前15行
lunr.Builder.prototype.add = function (doc, attributes) {
  var docRef = doc[this._ref],
      fields = Object.keys(this._fields)

  this._documents[docRef] = attributes || {}
  this.documentCount += 1

  for (var i = 0; i < fields.length; i++) {
    var fieldName = fields[i],
        extractor = this._fields[fieldName].extractor,
        field = extractor ? extractor(doc) : doc[fieldName],
        tokens = doc[fieldName + '_tokens'],
        terms = this.pipeline.run(tokens),
        fieldRef = new lunr.FieldRef (docRef, fieldName),
        fieldTerms = Object.create(null)
```

搜索定位替换以下部分
``` js
lunr.trimmer = function (token) {
  return token.update(function (s) {
    return s.replace(/^\s+/, '').replace(/\s+$/, '')
  })
}
```



### 通过MKDocs更新文档

* 使用命令 `git clone https://github.com/Yaluoo/docs-LowCodePlatform.git` 克隆资源库
* 使用VS Code编辑文档
* 使用Github Desktop提交变更


### 使用MKDocs

* 打开命令行将目录切换至文档所在目录
* 本地运行 `mkdocs serve`
* 更新网页 `mkdocs gh-deploy`
