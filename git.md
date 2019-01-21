# Git Tips & Tricks

Also checkout out https://github.com/git-tips/tips

## log

### View changes for a range of lines in a file
```sh
git log -L start,end:filepath
```

### Exclude changes from file
```sh
git log -- ':!filepath'
```

