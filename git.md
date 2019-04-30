在使用git的时候，有些文件是不需要上传的，所以就可以修改   .gitignore 

例如：
如果是对所有文件都取消跟踪的话，就是
git rm -r --cached . 　　//不删除本地文件
git rm -r --f . 　　//删除本地文件
对某个文件取消跟踪
git rm --cached readme1.txt    删除readme1.txt的跟踪，并保留在本地。
git rm --f readme1.txt    删除readme1.txt的跟踪，并且删除本地文件。
不要误解了 .gitignore 文件的用途，该文件只能作用于 Untracked Files，也就是那些从来没有被 Git 记录过的文件（自添加以后，从未 add 及 commit 过的文件）。