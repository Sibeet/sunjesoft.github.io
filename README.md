# sunjesoft.github.io

선재소프트의 Goldilocks Cluster 제품을 위한 가이드 및 Release 노트를 배포하기 위한 문서시스템을 수정하는 방법에 대해서 기술합니다. 


# 시작하기 

## Jekyll 설치 

본 사이트는 Jekyll 로 만들어졌습니다. 일단 본인의 PC 에 Jekyll 을 설치합니다.[Jekyll Install](http://jekyllrb-ko.github.io/docs/installation/) 에서 Jekyll 을 설치할 수 있습니다. 

윈도우 환경이라면 WSL ( Windows Subsystem for Linux)를 적극활용하는 것이 좋습니다. WSL 을 설정하는 방법은 [다음](https://joojy.net/p/20171224581)과 같습니다.
그 다음 bash 터미널에서 [다음](https://gist.github.com/CesarBMartinez/660230e7fad786b58a68cacf2cd709ad) 과 같은 방법으로 설치할 수 있습니다. 

## Fork 

자신의 github 계정으로 본 Repository 를 Fork 합니다. 

## Clone

Fork 한 Repository 를 로컬에 clone 합니다. 

```
$ git clone https://github.com/ckh0618/sunjesoft.github.io.git
```

이제 수정 후 commit 하고 push 한 후 PR (Pull Request) 하면 됩니다. 

직접 URL 링크를 통해서 수정하는 방법도 있습니다. 글을 읽다가 Edit 버튼을 누르면 바로 수정할 수 있습니다.

# 글 작성하기 

## 새로운 글 작성 

pages 하위 디렉토리에 guides 라는 곳에서 md 파일을 생성하고 글을 작성하면 됩니다. 글을 작성할때 다음과 같이 jekyll 을 띄워놓으면서 작업하면 즉시 글의 내용을 확인할 수 있어 좋습니다. 

```
$ bundle exec jekyll server
```

우와 같이 수행한 후 http://127.0.0.1:4000 으로 접근하면 로컬의 사이트로 접근할 수 있습니다. 글을 다 썼다면 다음과 같이 사이드 바에 링크를 달아줍니다. 해당 사이드 바는  _data/sidebars/guides_sidebar.yaml 파일을 열어보면 알 수 있습니다. 이 파일은 들여쓰기가 의미가 있는 파일이니 들여쓰기 잘해야합니다. 

## 이미지 경로 

이미지 파일은 중복을 피하기 위해서 /image/글의제목/ 에 위치하도록 합니다. 링크를 걸때도 /image/글의제목/1.jpg 형식으로 걸어주는 것이 좋습니다. 


