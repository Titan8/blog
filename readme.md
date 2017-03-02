## Building locally

To work locally with this project, you'll have to follow the steps below:

1. Install [Hugo](http://gohugo.io/)
2. Clone this project.  
  `git clone git@github.com:Titan8/blog.git`
3. Install theme.  
  `git submodule update --init`
4. Preview your project.  
  `hugo server -D`
5. Add content and start writing.  
  `hugo new post/title-of-your-essay.md`
6. remove draft state after review  
  `draft = true  # delete this line or change value to false`
7. Deploy it.  
  `./deploy.sh`
