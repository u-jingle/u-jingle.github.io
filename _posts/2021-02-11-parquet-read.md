---
title: "Big Data Tools Plugin으로 Parquet 파일 읽기"
layout: post
author: "JinggleJinggle"
header-style: text
tags:
  - parquet
  - BigDataTools
---
# 문제점
요즘 회사에서 우리팀은 파일의 용량을 줄이기 위해 json 파일을 parquet로 변환하여 s3에 저장해 사용하고 있다. (양이 맣은 경우에 한함)

parquet파일은 파일 자체를 바로 읽을 수 없기 때문에 잘 변환이 되었는지,, 또는 이슈가 생겨서 데이터를 확인해 봐야 하는 일이 생긴 경우에 나는 아래 두가지 방법으로 parquet 파일을 확인 해보곤했다.

1. 인터넷에서 이용할 수 있는 parquet reader를 사용한다. ([http://parquet-viewer-online.com/](http://parquet-viewer-online.com/) )
- 이 방법의 경우, 파일은 로컬에 저장시켜두고 사이트에 업로드만 하면 쉽게 확인이 가능하지만 파일의 용량이 10mb 이상인 경우에는 확인이 불가능하고, 그때그때 파일을 다운로드해서 사용해야하는 점이 굉장히 불편했다.
2. spark를 통해 read한다.
- 용량이 큰경우에도 가능하며, 내가 원하는 추가적인 로직을 통해 데이터를 확인할 수도 있지만 그때그때 코드를 작성하고 실행시켜 데이터를 확인해 보는 일은 굉장히 번거로웠다.

![https://jjalbot.com/media/2018/12/yffKuQeJe/zzal.jpg](https://jjalbot.com/media/2018/12/yffKuQeJe/zzal.jpg){: width="70%"} 

그러다 data tool로 볼수 있을 것 같은 생각이 들어 찾아 봤더니 역시 있었다..

jetbrain의 **Big Data Tools Plugin**으로 parquet 파일을 풀어 볼 수 있었다!!

# 해결방안
방법은 아래와 같다.

1.Datagrip을 실행시킨다.


2.preferences→plugins 에서 'Big Data Tools' 를 다운로드 받는다.
![/img/in-post/2021-02-11-parquet-read/Untitled.png](/img/in-post/2021-02-11-parquet-read/Untitled.png)


3.preferences→tools→Big Data Tools Connections에서 '+' 선택후 s3 를 선택하고 접근에 필요한 s3 정보를 입력한다.  
Test connection이 성공되면 적용한다.
![/img/in-post/2021-02-11-parquet-read/Untitled%201.png](/img/in-post/2021-02-11-parquet-read/Untitled%201.png)


4.그리고 나면 오른쪽에 Big Data Tools탭이 생기고 내가 입력한 s3 경로 하위의 partitioning과 그하위 파일들이 제대로 호출된 것이 확인된다.  
파일을 클릭하면 데이터를 확인 할 수 있다.
![/img/in-post/2021-02-11-parquet-read/Untitled%202.png](/img/in-post/2021-02-11-parquet-read/Untitled%202.png)  



위처럼 아주 간단하게 s3에 저장된 parquet파일을 확인 해 볼수 있었다.  

(나는 Datagrip으로 실행해 보았지만 IntelliJ에서도 동일한 방법으로 실행 가능하다.)

Big data Tools Plugin을 활용하면 s3외에도 아래와 같이 다양한 **파일시스템, 저장소, 모니터링 등의 기능**을 활용할 수 있다고 하니 유용하게 쓸수 있을 듯하다.  

![/img/in-post/2021-02-11-parquet-read/Untitled%203.png](/img/in-post/2021-02-11-parquet-read/Untitled%203.png){: width="30%"} 
