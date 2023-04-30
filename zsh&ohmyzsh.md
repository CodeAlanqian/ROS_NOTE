zsh与ohmyzsh



# zsh



```
sudo apt install zsh
```





# ohmyzsh

```
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```



# 主题









# 插件



## 自动补全 [zsh-autosuggestions](https://link.zhihu.com/?target=https%3A//github.com/zsh-users/zsh-autosuggestions)

```
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```



## 语法高亮 [zsh-syntax-highlighting](https://link.zhihu.com/?target=https%3A//github.com/zsh-users/zsh-syntax-highlighting)

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```





引入插件

```
nano ~/.zshrc
或
vim ~/.zshrc
```

找到**plugins = (git)**

```
plugins=(git zsh-syntax-highlighting)

//或者这样
plugins=(
    git
    zsh-syntax-highlighting
    zsh-autosuggestions
)
```

使用vim的话，按**i**进入插入模式，修改完成后按**ESC**，输入 **:wq** 保存退出



使修改生效

```
source ~/.zshrc
```

