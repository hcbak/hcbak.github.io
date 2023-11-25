---
title: Chirpy Jekyll Theme를 깃허브 Pages로 서비스하기
author: hcbak
date: 2023-11-26 04:30:25 +0900
categories: [Blog]
tags: []
img_path: /assets/img/posts/Blog/2023-11-26-Chirpy-Jekyll-Theme를-깃허브-Pages로-서비스하기/
---

매번 블로그를 하려다가 실패한다.

티스토리는 워드프레스로 옮긴다고 버렸고  
워드프레스는 느려서 버렸다.

내 포스팅은 어디 갔느냐?  
워드프레스 까지는 옮겼는데 이젠 못하겠다.  

그러고 있다가 블로그는 써야겠는데 깃허브도 써야겠는데 하면서 [Github Pages](https://pages.github.com/){:target="_blank"}를 찾아냈고 [Jekyll](https://jekyllrb-ko.github.io/){:target="_blank"}을 찾아냈고 [Chirpy](https://chirpy.cotes.page/){:target="_blank"}를 찾아냈다.

그냥 [Getting Started](https://chirpy.cotes.page/posts/getting-started/){:target="_blank"} 보고 정직하게 가면 될걸 구글링으로 쉬운 길 찾다가 너무 헤매서 적어놓는다. ㅂㄷㅂㄷ

## 정석

 [Getting Started](https://chirpy.cotes.page/posts/getting-started/){:target="_blank"}에 잘 작성되어 있으니, 보고 따라하면 된다.
 1. [**Chirpy Starter**](https://chirpy.cotes.page/posts/getting-started/#option-1-using-the-chirpy-starter){:target="_blank"} - 그냥 누르면 된다. 공식 문서에서 추천하는 방법이다.
 2. [**GitHub Fork**](https://chirpy.cotes.page/posts/getting-started/#option-2-github-fork){:target="_blank"} - 포크해다가 쓰는건데, 방법이 조금 귀찮다.

어쨌든 그냥 저거 보고 따라하면 쉬운데, 구글에 Chirpy 검색하면 zip파일 받아서 Jekyll 먼저 설치하고 안의 내용물을 덮어쓰는 방식으로 설명하는 사람들이 조금 있다. 잘 안된다. ㅂㄷㅂㄷ하다.

## 소스코드로 설치하기
그래서 결국 나는 소스코드로 설치했다. 테마를 만든 사람이 프론트를 node로 작성해서 빌드를 해주지 않으면 정적 파일들이 assets폴더에 들어가지 않는다.
결국 이걸로 오류나서 Actions가 뱉어버리는데, 위의 [**GitHub Fork**](https://chirpy.cotes.page/posts/getting-started/#option-2-github-fork){:target="_blank"} 방식이 이와 같다. 공식 문서에서는 **$ bash tools/init** 명령어로 해결하는데, init 파일 안에 보면 **npm i && npm run build** 명령어가 있다. init가 그냥 파일만 삭제한다고 적어놓으신 분들은 어찌하여 그렇게 적은 것인지 모르겠다. ㅂㄷㅂㄷ하다.

### 빌드 환경
그냥 Ubuntu로 했다.
```bash
sudo apt update
sudo apt upgrade

curl -sL https://deb.nodesource.com/setup_20.x | sudo bash -E -

sudo apt install ruby-full build-essential zlib1g-dev git nodejs

echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

gem install jekyll bundler

ruby -v
gem -v
jekyll -v
bundle -v
git --version
node -v
npm -v
```
Ubuntu 환경에서 작업할 때는 node를 위와 같이 최신 버전을 가져와서 설치해야 한다.  
최신인 Ubuntu 22.04 LTS에서도 아득한 옛날 버전을 들고 있어서 빌드하면 오류난다.

### 빌드
![01](01_Create-a-new repository.png)
_리포지토리 먼저 만들고 온다._
```bash
git clone https://github.com/hcbak/hcbak.github.io
git clone https://github.com/cotes2020/jekyll-theme-chirpy/

cd jekyll-theme-chirpy
bundle
bash tools/init
bundle lock --add-platform x86_64-linux

mv jekyll-theme-chirpy/* hcbak.github.io
```
커밋 하라고 귀찮게 하면 init를 열어서 그 부분을 날려버린다.
```
...
check_env() {
  _check_cli
  #_check_status ← 이거랑
  _check_init
...
main() {
  check_env
  checkout_latest_release
  init_files
  #commit ← 이거
...
```
대충 이렇게 하면 된다.
![02](02_npm-i-&&-npm-run-build.png)
_제일 중요한 npm i && npm run build_

### 깃허브 업로드
```bash
git add -A
git commit -m "Create page"
git push
```


![03](03_Settings-GitHub-Pages.png)
_Settings → Pages → Build and deployment → Source → GitHub Actions_


끝!

## 참고

[Jekyll(Ubuntu)](https://jekyllrb.com/docs/installation/ubuntu/){:target="_blank"}  
[Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy){:target="_blank"}  
[Chirpy Getting Started](https://chirpy.cotes.page/posts/getting-started/){:target="_blank"}  
[j1mmyson Blog](https://j1mmyson.github.io/posts/StartingBlog/){:target="_blank"}