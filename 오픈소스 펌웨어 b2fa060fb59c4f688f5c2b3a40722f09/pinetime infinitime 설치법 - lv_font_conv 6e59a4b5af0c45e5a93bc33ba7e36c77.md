# pinetime infinitime 설치법 - lv_font_conv

```
sudo rm -rf /usr/local/lib/node_modules
sudo rm -rf ~/tmp/node/node_modules
```

`$ sudo apt-get install build-essential libssl-dev
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
$ source ~/.bashrc
$ nvm --version
0.34.0

$ nvm install node
[...]
Now using node v12.6.0 (npm v6.9.0)
Creating default alias: default -> node (-> v12.6.0)

$ nvm use node
Now using node v12.6.0 (npm v6.9.0)

$ nvm run node --version
Running node v12.6.0 (npm v6.9.0)
v12.6.0`

1. Install lv_font_conv

`$ nvm run node --version
Running node v12.6.0 (npm v6.9.0)
v12.6.0

$ npm i littlevgl/lv_font_conv -g
$ lv_font_conv -v
0.0.1`