# 依赖内容

- NPM 18.14LTS

  > [Node.js (nodejs.org)](https://nodejs.org/en/)

- Hexo 5.4.0

  > [Hexo](https://hexo.io/)

- Hexo Aurora

  > [Home | Hexo Aurora (tridiamond.tech)](https://aurora.tridiamond.tech/zh/)

# 本地测试三步

```powershell
# 1. 
hexo clean
# 2. generate
hexo g
# 3. start server
hexo server
```

# 提交远程仓库

```powershell
# 提交到master分支以在https://JBR-Bunjie.github.io页面展示结果
hexo d
# 提交到hexo-source分支以在github仓库中保存文档与配置
git push -u origin hexo-source
```