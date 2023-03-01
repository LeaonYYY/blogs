# 解决: brew安装软件报“no bottle available”的错误

macOS在使用homebrew安装软件时:命令行报错

```bash
Error: git: no bottle available!
You can try to install from source with:
  brew install --build-from-source git
Please note building from source is unsupported. You will encounter build
failures with some formulae. If you experience any issues please create pull
requests instead of asking for help on Homebrew's GitHub, Twitter or any other
official channels.

```

解决办法:

```bash
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
brew update
```

更新完成后再下载就可以了