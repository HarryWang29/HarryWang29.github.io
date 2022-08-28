---
title: "突破vuepress Theme Reco 1.x的密码限制"
date: 2022-08-26T22:07:01+08:00
description: "一次有趣的挑战"
tags: 
  - "security"
  - "vue"
---

## 起因
在这个blog准备架设的时候，查看了一个前端同事的blog，他的架设方式为：
* github
* action
* vurpress

在观察他blog的过程中，他提到了有一个文档是属于加密状态，此时作为一个后端开发，我立刻想到了一个问题，你一个纯前端页面，如何能够保证自己的密码安全？

## 挑战
在我提出质疑之后，前端同事向我提出了挑战，让我看看是否能够看到里面的内容，作为一个喜欢瞎折腾的人，必然接受了挑战

## 解决方案
### github-master
由于他是基于github+action，那么我非常简单的就找到了文本内容，步骤如下：
* 打开仓库首页
* 确认master分支
* 找到文档存储位置
* 点开文档

好了，非常简单，因为action的原因，你必须要将自己的文档完全的放入仓库中，action才能进行编译等操作
![CleanShot 2022-08-28 at 23 41 40@2x](https://user-images.githubusercontent.com/8288067/187082498-2a0dc5c3-f433-4570-8249-bde2094738f3.png)

但是此时看到内容，我总感觉没啥意思，好了，开始给自己加活

### github-gh-pages
其实这个方法的原理和master的方法一样，都是有源码的情况下，能够看到静态的html，流程如下：
* 打开仓库首页
* 确认`gh-pages`分支
* 找到文档存储位置
* 点开文档

依旧非常简单，我们在html中看到了文本的正文

![](https://user-images.githubusercontent.com/8288067/187082671-88dbf5c0-b58b-4f43-b948-d862bdae4316.png)


以上两种其实算是我知道他源码仓库的情况下，假设他修改了blog域名，我不知道他的github仓库，那么我要如何操作呢？

### chrome->view page source
祭出`chrome`（其实firefox什么的都一样，没有什么复杂的东西）
1. 打开blog加密页
    
    界面如下
    ![](https://user-images.githubusercontent.com/8288067/187082968-3c54edb7-febe-4d70-8cda-5881237dc5dd.png)

2. view page source
   
   到这一步大家应该发现了，其实这里的page和上面的`gh-pages`中一样了，html文件中写着正文

其实到这里我本来不想再进行下去了，不过一直没有正式的把`Konck!Knock!`干掉总也有点不甘心

### chrome替换js大法
查看[vuepress-theme-reco-1.x](https://github.com/vuepress-reco/vuepress-theme-reco-1.x/blob/f5d6cbc6d7bd74808392fcea8bc028104e2dfc95/packages/vuepress-theme-reco/components/Password.vue)关于密码部分源码，发现密码的校验与生成也是非常简单
```js
    const isHasPageKey = () => {
      const pageKeys = instance.$frontmatter.keys.map(item => item.toLowerCase())
      const pageKey = `pageKey${window.location.pathname}`
      return pageKeys && pageKeys.indexOf(sessionStorage.getItem(pageKey)) > -1
    }
    const inter = () => {
      const keyVal = md5(key.value.trim())
      const pageKey = `pageKey${window.location.pathname}`
      const keyName = isPage.value ? pageKey : 'key'
      sessionStorage.setItem(keyName, keyVal)
      const isKeyTrue = isPage.value ? isHasPageKey() : isHasKey()
      if (!isKeyTrue) {
        warningText.value = 'Key Error'
        return
      }
      warningText.value = 'Key Success'
      const width = document.getElementById('box').style.width
      instance.$refs.passwordBtn.style.width = `${width - 2}px`
      instance.$refs.passwordBtn.style.opacity = 1
      setTimeout(() => {
        window.location.reload()
      }, 800)
    }
```

根据源码，我们发现，其实他把用户输入的密码经过md5计算之后，存入到`sessionStorage`中，然后再与文件中`key`的值进行比较，若存在则通过，不存在则提示`Key Error`，我想如果思路活跃的同学，应该已经有一些想法出现了，没错，既然你是本地比对，那么我把你的基准值修改了呢？

1. chrome -> f12 -> source -> page
2. 保存`app.xxxx.js`，注意，此步骤保存的话，一定一定要与`page`中的路径完全一直才行

    ![CleanShot 2022-08-29 at 00 06 54@2x](https://user-images.githubusercontent.com/8288067/187083567-7d71e976-ef16-475d-bae1-0847388a390d.png)
    ![CleanShot 2022-08-29 at 00 07 51@2x](https://user-images.githubusercontent.com/8288067/187083619-11dda1f9-3bc4-44c2-9d7c-d12d7a210d11.png)
3. 准备一个md5-hash
   ```bash
   md5 -s 1
   # MD5 ("1") = c4ca4238a0b923820dcc509a6f75849b
   ```
4. 搜索替换hash，找到app.js中现有的hash，替换为`c4ca4238a0b923820dcc509a6f75849b`
5. chrome -> f12 -> source -> Overrides(此处要注意，默认这个看不到，点击`Filesystem`边上的`>>`里面有)
6. 点击`Select folder for overrides`选择创建的`top`文件夹
7. 此处要注意一下，由于我当时正好是竖屏操作的，在屏幕上方会有一个权限验证，我当时可是找了好久这个验证在哪里

   ![CleanShot 2022-08-29 at 00 15 53@2x](https://user-images.githubusercontent.com/8288067/187083965-5945ff2a-b4ad-41c7-83a4-71daeabd4c52.png)
8. 点击`Allow`
9. 在`overrides`中，就能够看到被替换的js文件了，展示了一个小点，说明已经替换了

    ![CleanShot 2022-08-29 at 00 17 52@2x](https://user-images.githubusercontent.com/8288067/187084043-48563d5e-b54d-48c3-ad0f-95df7b01e072.png)
10. 刷新页面，将我们替换的js重新加载
11. 输入密码`1`
12. `Key Success`

    ![](https://user-images.githubusercontent.com/8288067/187084173-194fab33-09f6-4eff-a9d9-7bac094d6c42.png)

## 总结
其实此次的研究没有什么实际的内容，不过就是证明了，前端自身做安全验证，是真的没有任何可靠性