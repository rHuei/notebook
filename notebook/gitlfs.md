# 2019 1月份 SA@Tainan 1/20(日) GitLab 與 Git LFS 介紹
簡報:
- [git-lfs](/files/Git LFS.pptx)
- [git-lfs](http://gitlfs.rsync.tw)
- [.dev](http://gandi-dev.rsync.tw)
# demo
當被加入lfs檔案會將大檔案宣告為binary檔案，要使用lfs方式上傳
```bash
touch .gitattributes
git lfs track test.psd
```
#### 可以建立branch保護資料
```bash
git checkout -b prod
git add file.iso
git merge master
CI push prod branch to productio
```
# Git Attributes
- Identifying Binary Files
- Customize each file process when git show/diff/checkout/commit

- Create a .gitattributes in your project directory or,
- Save in .git/info/attributes then you don’t need .gitattributes( also don’t need to commit )
# Customizing - Diffing Binary Files
如果檔案是docx在git是無法被比對的，可以透過以下方式作強制比對
#### docx
```bash
docx(http://docx2txt.sourceforge.net/):
*.docx diff=word
$ git config diff.word.textconv docx2txt
```

#### 圖片
```bash
*.png diff=exif
$ git config diff.exif.textconv exiftool
```

# 自訂code
如果想要git可以自動顯示上傳日期，可以自己設定變數直接自動顯示日期
```bash
$ git config filter.dater.smudge expand_date
$ git config filter.dater.clean 'perl -pe "s/\\\$Date[^\\\$]*\\\$/\\\$Date\\\$/"'

epeand_date.ruby:
#! /usr/bin/env ruby
data = STDIN.read
last_date = `git log --pretty=format:"%ad" -1`
puts data.gsub('$Date$', '$Date: ' + last_date.to_s + '$')
```
#### example
a1.py:
```
# $Date$
import …
abc = “abc”
for ...
```
```bash
$ git checkout a1.py
$ cat a1.py
# $Date: Tue Apr 21 07:26:52 2009 -0700$
import …
abc = “abc”
for...
```
