# 自动发布测试

### auto-release-module-a

<a href="https://badge.fury.io/js/auto-release-module-a" title="npm">
<img src="https://img.shields.io/npm/v/auto-release-module-a.svg?style=flat-square" alt="npm version"/>
</a>

### auto-release-module-b

<a href="https://badge.fury.io/js/auto-release-module-b" title="npm">
<img src="https://img.shields.io/npm/v/auto-release-module-b.svg?style=flat-square" alt="npm version"/>
</a>

### 测试场景

- [x] 子包1 bug fix, 子包2 无改动, 只发子包1
- [x] 子包1 bug fix，子包2 feat， 子包1 0.0.x  子包2 0.x.0
- [x] 子包1 和 子包2 同时 bug fix, 同时发包, 两个包单独的 commit，两个单独的 release
![image](https://user-images.githubusercontent.com/21015895/137615464-c86ce270-5b43-463b-9a98-3bf5aa9b74be.png)
![image](https://user-images.githubusercontent.com/21015895/137615575-4d012c44-3fdf-4028-92da-e1f3c3798b01.png)


- [x] 父级 有 fix feat 更新, 子包1，子包2 的 changelog 都会含有 父级的更新
- [x] 相关联的 pr 会有 release 提示
- [x] master 为保护分支，通过 pr 的形式 `chore(release): x.x.x` 合并到 master，能正常触发 action 并且推送 changelog 和 tag，release
- [x] 可以自动生成 release， 自动添加 changelog， 多个子包生成不同的 release，标题为 （包名-版本号）

![image](https://user-images.githubusercontent.com/21015895/137594106-2e7abba2-2b8e-4a72-8b64-5ba0722dbfdb.png)
![image](https://user-images.githubusercontent.com/21015895/137594255-b460d4a8-bf20-42c4-9c18-8686f8b52dc5.png)

- [x] release 后 钉钉群通知： 由于 release 是机器人执行的, action 监听 release published 触发不了, 我的做法是 放在 release 后执行

```
  - name: Release
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
    run: yarn release

  - name: Release notify
    if: ${{ success() }}
    uses: visiky/dingtalk-release-notify@main
    with:
      DING_TALK_TOKEN: ${{ secrets.DING_TALK_GROUP_TOKEN }}
      notify_title: '🎉 {release_tag} 发布 🎉'
      notify_body: '## { title }<hr /> ![preview](https://gw.alipayobjects.com/zos/antfincdn/WWAv%24ctWd/da21d637-2232-4a64-a994-c0227ab759e2.png) <hr /> { body } <hr />'
      at_all: false
```
![image](https://user-images.githubusercontent.com/21015895/137616518-81ceae08-6d15-4900-85d5-217282868859.png)



### 问题

1. 发布时会根据当前的 commit 是 feat 还是 fix 决定 是 `0.0.x` 还是 `0.x.0` 发布到 npm, 但是不会更新 package.json 需要在 `chore(release): x.x.x` 所对应的pr 手动修改一下版本号

![image](https://user-images.githubusercontent.com/21015895/137594169-858c7bef-d890-4f90-bef4-1e8591982394.png)
![image](https://user-images.githubusercontent.com/21015895/137594174-97131cd7-03f6-4603-9957-49bf214f59fd.png)

2. 测试 子包 都为 `0.0.0` 首次发布默认会更新为 `1.0.0` 感觉**首次需要手动指定版本**
3. 如何让 semantic release 可以正常触发 release notify action ? 默认的 GITHUB_TOKEN 是 github 的一个虚拟账号

```yml
name: 🎉  Release Notify

on:
  release:
    types: [published]
```

![image](https://user-images.githubusercontent.com/21015895/137614074-f6dc253c-0f04-40b3-bfdd-6f6bddb08871.png)

如果 action 监听 release published，github 默认 token 是不会触发的, 认为是机器人执行 防止套路循环执行
![image](https://user-images.githubusercontent.com/21015895/137614111-81e0ec34-5c07-46b4-a1d2-6da18c222231.png)


而 semantic 所使用的 GITHUB_TOKEN 是一个真实的 github 账户
![image](https://user-images.githubusercontent.com/21015895/137614047-92fc69a2-c714-4419-bc07-a3a44394c8a6.png)

![image](https://user-images.githubusercontent.com/21015895/137614051-662556d8-f304-4786-a6a8-e3bbd712f2bf.png)

目前测试下来没有按预期的触发

![image](https://user-images.githubusercontent.com/21015895/137616585-67bb37b4-b7ee-469d-8e63-9b58c8b5e37c.png)

如果能把 release 时候的 token 替换成 semantic token 也许可以触发 action 的 release published



