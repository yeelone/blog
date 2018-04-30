## Mac 安装polipo实现终端代理

一直用shadowsock，但go get 不支持sock5，所以需要借用polipo来实现终端代理

### polipo 
#### 安装

```
✗ brew install polipo

```


### polipo转换协议

```
✗ polipo socksParentProxy=127.0.0.1:1086
```

### 配置zsh
```
 ✗ vim ~/.zshrc 
export http_proxy="http://127.0.0.1:8123/"
export https_proxy="http://127.0.0.1:8123/"
export no_proxy="localhost,127.0.0.1"
export HTTP_PROXY="http://127.0.0.1:8123/"
export HTTPS_PROXY="http://127.0.0.1:8123/"
export FTP_PROXY="http://127.0.0.1:8123/"
export NO_PROXY="localhost,127.0.0.1,localaddress"

✗ source ~/.zshrc

```

##### 测试网络状态
```
	✗ curl www.google.com
```


### 启动开机
```
 ✗ ln -sfv /usr/local/opt/polipo/homebrew.mxcl.polipo.plist ~/Library/LaunchAgents

✗ vim /usr/local/opt/polipo/homebrew.mxcl.polipo.plist
<plist version="1.0">
  4   <dict>
  5     <key>Label</key>
  6     <string>homebrew.mxcl.polipo</string>
  7     <key>RunAtLoad</key>
  8     <true/>
  9     <key>KeepAlive</key>
 10     <true/>
 11     <key>ProgramArguments</key>
 12     <array>
 13       <string>/usr/local/opt/polipo/bin/polipo</string>
 14       <string>socksParentProxy=localhost:1086</string>   增加了这一行
 15     </array>
 16     <!-- Set `ulimit -n 65536`. The default macOS limit is 256, that's
 17          not enough for Polipo (displays 'too many files open' errors).
 18          It seems like you have no reason to lower this limit
 19          (and unlikely will want to raise it). -->
 20     <key>SoftResourceLimits</key>
 21     <dict>
 22       <key>NumberOfFiles</key>
 23       <integer>65536</integer>
 24     </dict>
 25   </dict>
 26 </plist>
```