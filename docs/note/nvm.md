# nvm Node版本工具

## 安装
```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

$ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

## 配置环境变量
```shell
# zshrc
# 1、这是本地不存在配置文件的时候提示需要添加的配置
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm

# 2、这是本地存在配置文件的时候提示需要添加的配置（推荐）
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

载入配置文件
```shell
source ~/.zshrc
```

## 遇到的问题
1. 执行nvm命令，找不到命令，是环境变量配置、配置载入造成的。
2. nvm install 下载源过程慢，可以切淘宝源提速 在~/.bashrc文件中添加export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
