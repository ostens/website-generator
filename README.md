# Website generator
### Adding a new post to website
* **Always check `git status`**
* Change directory to the quickstart folder
* Create new post using hugo:
```bash
hugo new post/post-name.md
```
* Add content to the new post
* Run hugo server to preview the changes:
```bash
hugo server
```
* Compile changes:
```bash
hugo
```
* Change directory to the public folder
* Check git status:
```bash
git status
```
* Add all changes:
```bash
git add .
```
* Commit changes:
```bash
git commit -m "comment"
```
* push changes to remote repository
```bash
git push
```

Remember that this will only update the website repository and not the website generator repository! cd to the quickstart folder and repeat steps that were done in the public folder (the web repository).
