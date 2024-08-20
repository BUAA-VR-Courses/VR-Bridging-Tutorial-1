## gitbook 环境

nvm 管理 nodejs 版本，对 gitbook 来说 v10 及以下问题比较少

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
nvm install 10
nvm use 10.24.1
```

安装 gitbook

```bash
# npm config set registry http://mirrors.cloud.tencent.com/npm/
npm install gitbook-cli -g
gitbook -v	# 自动下载gitbook
```

安装插件

```bash
gitbook install
```

构建和运行

```bash
gitbook build
gitbook serve
```

