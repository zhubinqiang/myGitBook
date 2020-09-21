# git 结合 batch来clone和pull

```
set REMOTE=bzhux@media-pxe3:~/WS/repos


call :cloneOrpull "wiki"
call :cloneOrpull "workJournal"
call :cloneOrpull "OSConf"
call :cloneOrpull "myGitBook"
call :cloneOrpull "myWorkBook"
pause


:cloneOrpull
if exist %1 (
    cd %1
    git pull origin master
    cd ..
) else (  
    echo "git clone %REMOTE%/%1"
    git clone %REMOTE%/%1
)
goto:eof
```

