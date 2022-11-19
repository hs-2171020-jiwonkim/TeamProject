# 시스템 설계서

## 캠퍼스 단위 개인화 커뮤니티

- 문서 갱신일 : 2022/11/13

---

# 목차

1. 개요
   - 서비스의 목적
   - 서비스 설명
   - 서비스 구조
2. 유사 서비스 분석
3. 사용된 오픈소스 소프트웨어

### Elastic Search

#### 설명

Apache Lucene을 기반으로 만들어진 오픈소스 소프트웨이며, 데이터를 검색, 저장, 분석하는데 유용한 도구이다.  
특히, 데이터 검색이 관계형 데이터 베이스에 비해 굉장히 빠른데 그 이유는 역색인을 통해 데이터를 저장하고 검색하기 때문이다.  
그렇기 때문에, 데이터 베이스에 저장된 데이터의 양과 관계없이 검색 속도는 일정하다.  
Elastic Search는 정형 데이터와 비정형 데이터를 모두 받아 들일 수 있으며, Kibana를 통해 검색 결과를 시각화할 수 있다.  
거기에 더해 검색 결과를 그냥 내보내는 것이 아니라 관련도가 높은 데이터를 우선적으로 보여준다.  
따라서 Elastic Search를 단순한 검색 엔진이 아닌 초고속 데이터 베이스 이면서 빅데이터 플랫폼으로 받아들이는 것이 더 올바르다.

#### 입출력데이터의 형식

기본적으로 Elastic Search는 모든 형태의 데이터에 대하여 쿼리를 할 수 있다.  
(쿼리란 데이터 베이스의 데이터를 변화시키지 않고 속성 데이터를 조사하는 것을 의미한다.)  
하지만, 이 플랫폼에서는 텍스트를 위주로 입력과 출력이 이루어질 예정이다.

#### 특징

1. Scale out  
   규모가 수평적으로 늘어날 수 있다.

2. 고가용성  
   데이터의 안정성을 높여준다.

3. Schemaless  
   Json 문서를 통해 데이터를 검색하므로 스키마가 필요 없다.

4. 역색인
   본문의 검색어를 먼저 추출한 뒤 검색어에 해당하는 문서(document)를 찾는다.

5. 실시간성  
   데이터 분석이 실시간으로 이루어지며 사용자가 원하는 결과를 바로바로 보여줄 수 있다.

#### 관계형 데이터 베이스를 계속 쓰는 이유

Elastic Search는 관계형 데이터 베이스가 아니기 때문에 완전한 대체는 할 수 없다.  
Elastic Search가 더 나은 영역이 있고, 관계형 데이터 베이스가 더 나은 영역이 있다.  
데이터가 빈번하게 업데이트 되는 상황이라면 관계형 데이터 베이스를 사용하는 것이 더 좋을 것이고,  
데이터의 삭제없이 로그성으로 남겨야 하는 상황이라면 Elastic Search가 더 유용하다.

#### 작동방식

Elastic Search는 단독으로 사용되기도 하고, ELK(Elastic Search/Logstash/Kibana) 스택으로 쓰이기도 한다.

##### Logstash

다양한 소스(DB, CSV 파일등)의 로그 또는 트랜잭션 데이터를 수집, 집계, 파싱하여 Elastic Search에 전달

##### Elastic Search

Logstash로 부터 받은 데이터를 검색 및 집계를 하여 필요한 정보를 획득

##### Kibana

Elastic Search의 빠른 검색을 통해 데이터를 시각화 및 모니터링

#### 시스템 구조와 데이터 구조

1. 클러스터

독립된 Elastic Search 시스템 환경으로, 1개 이상의 노드로 구성되며 각각의 클러스터는 내부적으로 데이터의 교환이 이루어지지 않는다.
하나의 클러스터를 다수의 서버가 구성할 수도 있고, 다수의 클러스터를 하나의 서버가 구성할 수도 있다.

2. 노드

실행중인 Elastic Search 시스템의 프로세스이다. 검색을 할 때 쿼리를 요청 받은 노드를 Coordinate 노드라고 부르며, Coordinate 노드는 각각의 샤드(다른 노드에 있는 샤드들 포함)들에게 검색 명령을 내린다. 샤드별로 검색이 분산 실행되고, 검색 결과를 다시 Coordinate 노드에게 전달한다. 검색 결과를 받은 Coordinate 노드는 자신이 받은 결과를 취합하여 사용자에게 응답한다.

3. 도큐먼트

Elastic Search에 저장되는 단일 데이터 단위.
사용자는 어떤 도큐먼트가 어떤 샤드를 가지는 지 알 수 없고 알 필요도 없다.

4. 샤드

색인&검색을 진행하는 작업 단위이며, Primary 샤드와 Replica 샤드로 나뉘어진다. 샤드는 클러스트내의 다른 노드에 분산되어 저장됨으로써 데이터의 신뢰성을 가진다.

5. 인덱스

도큐먼트의 논리적 집합으로, 1개 이상의 샤드로 구성된다.
인덱스 별로 Primary 샤드와 Replica 샤드의 세트 수를 설정한다.
Primary 샤드의 수는 처음에 설정하고 나면 수정이 불가능하지만 Replica 샤드의 수는 수정이 가능하다.

인덱스를 만드는 코드

<pre>
<code>
PUT books // books 라는 인덱스를 만듦
{
   "settings":{
      "index":{
         "number_of_shards": 5, //Primary 샤드의 개수를 5개로 설정
         "number_of_replicas": 1 //Replica 샤드의 개수를 1세트로 설정
      }
   }
}
</code>
</pre>

#### 참고

Datastream - 새로운 Elasticsearch 데이터 구조 이해하기 : https://youtu.be/JqKDIg8fgd8

[Elasticsearch] 기본 개념잡기 : https://victorydntmd.tistory.com/308

[ Elasticsearch ] 엘라스틱 서치 가볍게 살펴보는 개념 :) : https://youtu.be/MWItWo67F14

데이터 분석 플랫폼의 새로운 트렌드 "엘라스틱서치"(엘라스틱서치코리아 김관호 상무) : https://youtu.be/nOFB3jTnHEk

깃허브 주소 : https://github.com/elastic/elasticsearch

### Scrapy

#### 설명

Scrapy는 웹 사이트를 돌아다니면서 구조화된 데이터를 추출하기 위해 파이썬으로 작성한 오픈소스 프레임워크이다.
추출한 데이터를 데이터 마이닝, 정보 처리, 이력 기록 등 일련의 유용한 애플리케이션에 활용할 수 있다.
스크래피는 가볍고, 빠르고 확장성이 좋으며 비동기 네트워킹 라이브러리(synchronous networking library)인 Twisted를 기반으로 하기 때문에 매우 우수한 성능을 발휘한다.
XPath, CSS 표현식으로 HTML 소스에서 데이터 추출이 가능하고 현재 스크래핑허브(Scrapinghub), 플래스(Flax) 고스크레이프(GoScrape) 등 많은 기업들이 상용 지원을 제공하고 있다.

#### (Crawling)크롤링 이란

웹에는 수억개의 웹페이지가 있으며, 대부분의 페이지들은 수많은 정보를 가지고 있다.
최근 빅데이터가 대두되면서 이전에 작성되었던 페이지에서 유의미한 정보를 도출하기 위한 여러 가지 방법들이 논의되고 있는데 이를 Scraping 혹은 Crawling이라고 한다.

#### 입출력데이터의 형식

Scrapy는 웹 페이지에 있는 텍스트나 이미지 등 다양한 형태의 데이터들을 크롤링 할 수 있고 이 플랫폼에서는 주로 텍스트 형식의 데이터를 입출력한다.

#### 특징

1. Python 기반의 프레임워크로 쉬운 설정이 가능하다.

2. 단순한 스크랩 과정덕분에 크롤링 후, 바로 데이터 처리가 가능하다

3. Xpath, CSS표현식으로 HTML소스에서 데이터 추출이 가능하다.

4. webdriver를 사용하지 않는다

5. 다른 크롤링 오픈소스에 비해 가볍고 빠르다.

#### 구조

Scrapy의 아키텍처 구조에는 수집 주기를 설정하는 Scheduler가 존재하고 수집할 항목을 정의하는 Item과 수집 데이터의 저장 형식을 정의하는 Pipeline이 출력을 담당하는 형태를 가지고 있다.
그리고 Spiders를 통해 웹페이지의 정보를 수집한다.

1. Scheduler

스케쥴러는 수집 주기, 프록시 설정, 멀티 에이전트 설정 기능을 가지고 있어 Scrapy 엔진의 수집에 관련된 정책 사항을 설정하는 역할을 담당한다.

2. Item Pipeline

아이템 파이프라인은 수집하려는 데이터의 입출력을 담당한다. 수집하려는 항목을 아이템으로 정의하고 수집한 데이터의 형태를 파일 혹은 DBMS로 직접 입력이 가능하도록 설정할 수도 있다.

3. Spiders

수집하는 데이터를 크롤링하는 역할을 한다. 스파이더는 스케쥴러로부터 프로젝트에서 크롤링하는 정책에 따라 설정값을 요청하여 다운로더로부터 받은 크롤링 데이터를 아이템의 형태로 아이템 파이프라인에 전송한다.

4. Downloader

http, ftp 프로토콜을 해석하여 웹에 있는 데이터를 다운로드하는 역할을 담당한다.

#### 설치 및 실행 방법

##### Scrapy 설치

pip가 설치되어 있는 쉘에서 아래와 같은 명령어를 입력하여 Scrapy를 설치한다.
$ pip install scrapy

##### 프로젝트 생성 및 프로젝트 구조

scrapy라는 명령어를 통해 프로젝트를 생성하거나, 작성된 프로젝트를 실행할 수 있다.
$ scrapy startproject project명

##### item.py

크롤링해오는 데이터를 class로 받아오는 파일로 가지고 올 element를 작성한다.

ex)
import scrapy
from scrapy.item import ITem, Field
class APT2UItem(scrapy.Item):
depname=scrapy.Field() //단과대 element
trackname=scrapy.Field() //트랙명 element
trackcontent=scrapy.Field() //트랙의 설명 element

##### spider.py

데이터 수집 절차에 대한 수행 코드를 정의하는 파일로 해당 웹페이지에서 추출하고자 하는 정보들의 위치와 정보의 구조를 파악한다.
그리고 정규표현식을 이용하여 확보한 전체 웹페이지 정보 중 필요 데이터만 추출한다.

##### pipeline.py

파일, DB, 이메일 발송 등 수집된 데이터 처리 방식을 정의한다. 한글 처리를 위해 저장할 파일에 대한 utf-8 설정이 필요하다.

##### settings.py

프로젝트 모듈간 연결 및 기본 설정을 정의하는 파일이다.

#### DB 와 Scrapy 연결 방법

파일로 저장하는 방법과의 차이점은 Scrapy의 프로젝트 명을 APT2U에서 APT2U_DB로 변경한다는 점과 pipelines.py의 파일이 다르다는 점이다.
DB로 저장하기 위해서 크롤링의 결과를 저장할 Table을 생성해야 한다.

Scrapy 프로젝트 명 변경 ex) scrapy crawl APT2U_DB

"hs" Table 생성 스크립트 예시
CREATE TABLE `hs` (
`depname` varchar(200) DEFAULT NULL,
`trackname` varchar(200) DEFAULT NULL,
`trackcontent` varchar(500) DEFAULT NULL,
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

#### 참고

Scrapy github : https://github.com/scrapy/scrapy

[Scrapy] 웹사이트 크롤링해서 DB 저장 하기 : https://uslifelog.tistory.com/54

(파이썬) Scrapy를 이용한 웹 데이터 크롤러(Crawler) 만들기 : https://blog.naver.com/PostView.nhn?blogId=rjs5730&logNo=221280231854&categoryNo=14&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=search

[티스토리 게시글 추천 시스템 만들기] #5 Scrapy로 스크랩하기 : https://weejw.tistory.com/536

How to Connect Scrapy to your PostgreSQL database : https://nicolas-bourriez.medium.com/how-to-connect-scrapy-to-your-postgresql-database-6d27230ec6f8

대용량 데이터를 수집하기 위한 생산성있는 웹 크롤러의 구조 : https://exmemory.tistory.com/81

4. Data-Flow-Diagram

---

### <IGListKit>



#### 설명



IGListKit은 빠르고 유연한 목록을 작성하기 위한 데이터 중심 UICollectionView 프레임 워크이다. 

IGListKit은 UICollectionView(여러 데이터를 관리하고 커스텀 할 수 있는 레이아웃을 사용해서 사용자에게 보여줄 수 있는 객체)에 표시할 개체의 배열을 제공해준다.



#### 특징

디핑: 가장 긴 공통 하위 시퀸스(*longest common subsequence*) 기술을 사용하여 선형 시간에서 컬렉션 간의 최소 차이를 찾는다. 이때의 시간복잡도는 O(n)이다. 데이터 배열 사이에서 모든 삽입, 삭제, 업데이트 및 이동을 찾는다. 따라서 사용자들이 좋아요를 누른 수를 실시간으로 업데이트 해줄 수 있다. 다음은 이를 구현하기 위한 코드 중 일부이다.

```
extension User: ListDiffable {
  func diffIdentifier() -> NSObjectProtocol {
    return primaryKey
  }

  func isEqual(toDiffableObject object: Any?) -> Bool {
    if let object = object as? User {
      return name == object.name
    }
    return false
  }
}
```



#### 구조

1. 섹션 컨트롤러 만들기

새로운 섹션을 만들기 위해 IGListSectionController를 참조한다. 'cellForItemAtIndex:``sizeForItemAtIndex:'를 이용하여 재정의 해준다.

2. UI 만들기 

하나 이상의 섹션 컨트롤러를 생성후 IGListAdapter를 생성해준다.

3. 데이터 소스 연결

UI를 만들어 준 후 IGListAdapter의 데이터 소스 일부와 데이터를 반환해준다.

4. 최상위  Post 모델 설계

```
final class Post: ListDiffable {
  // 1
  let username: String
  let timestamp: String
  let imageURL: URL
  let likes: Int
  let comments: [Comment]
  // 2
  init(username: String, timestamp: String, imageURL: URL, likes: Int, comments: [Comment]) {
    self.username = username
    self.timestamp = timestamp
    self.imageURL = imageURL
    self.likes = likes
    self.comments = comments
  }
}
```

ListDiffable 프로토콜을 준수하는 Post모델이다. 여기서 주목할 점은 ListDiffable 프로토콜을 준수한다는 점이다.

```
func diffIdentifier() -> NSObjectProtocol {
  return as NSObjectProtocol
}
func isEqual(toDiffableObject object: ListDiffable?) -> Bool {
  return Boolean
}
```

이것이 피드에 관심도 별로 글을 정렬하기 위해(상단으로) 필수적인 것이다. LTR에서 정보를 받아와 해당 프로토콜을 통해 사용자의 관심도에 따라 글의 우선순위를 정렬해준다. 글에 대한 추천수를 실시간으로 받을때마다 서버로부터 새로운 데이터를 전달받는다. 이를 충돌없이 구현하기 위한 기능이다.



#### 참고

diffing : https://instagram-engineering.com/open-sourcing-iglistkit-3d66f1e4e9aa



IGListKit구조 : https://leejigun.github.io/IGListKit
