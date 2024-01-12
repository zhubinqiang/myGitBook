# screen


```bash
screen -S session_name

screen -ls

ctrl + a + d # deattach
ctrl + a, ctrl + d ## detache

screen -r session_name

ctrl + a + c # create a new tab
ctrl + a + x # exit a tab
ctrl + a + n/p (next, previous)

ctrl + a + S ## split the screen into up and down
ctrl + a + Tab ## switch to up or down

ctrl + a + x ## close

# scrolling operations are like in tmux
ctrl + a + [ ## copy mode, scroll screen
space        ## copy
enter        ## finish copy
ctrl + a + ] ## paste 
```

[^tips]

[^tips]: https://zhuanlan.zhihu.com/p/42551093