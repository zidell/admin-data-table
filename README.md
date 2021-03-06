# admin-data-table
어드민 관련 페이지를 만들려면 꼭 필요한 것이 두 가지 있다. 바로 데이터를 시각화한 차트와 실제 데이터를 보여주는 데이터테이블이다. 차트는 짱좋은 http://www.highcharts.com/ 를 쓰면 되는데 문제는 데이터테이블이 간단한 것 같으면서도 막상 하려면 손이 많이가는 아주 귀찮은 존재이다. 일단  `table` 태그를 이용하여 대충 테이블을 만들어 놓을 순 있지만, 실제로 서비스를 운영하다보면 되는게 없어서 자연스럽게 DB에 직접 접속을 하게 된다. 유명한 https://datatables.net 을 써도 무방하지만 기본적으로 클라이언트단에서 테이블 네비게이션이 일어나기 때문에 어드민 페이지에 어울리는 대용량 데이터를 처리하기엔 애매한 구석이 있다.

## HTML Table의 단점
기본적인 테이블을 쓰는 경우에는 열람 이외에 보다 더 자세한 작업이 불가능하다. 한마디로 정적인 인터페이스다. 물론 구석에 수정 및 삭제 링크를 두어 추가 작업을 하도록 여지를 둘 순 있지만, 무엇보다 원하는 데이터로의 접근이 썩 편하지가 못하다. 이를테면 각 컬럼별 정렬 조건을 추가하거나 특정 컬럼의 특정값으로 검색하는게 여의치 않다(보통의 경우에 테이블 외에 검색 폼을 두어 조건과 값을 입력하도록 되어 있지만 직관성이 부족하고 AND 연산폼을 구현하는게 쉽지 않다). 쌩 테이블의 후달리는 점들을 개선해서 나오는 것들이 아래의 DataTable.js 등이다.

## DataTable.js의 단점
일단 이 라이브러리는 굉장히 편하고 빠르고 좋다. 보통의 경우에는 이 라이브러리를 통해서 간단하게 해치울 수 있을 것이다. 하지만 관리자단에서 특정 테이블을 관리한다는 것은 몇 십만 건~몇 천만 건의 데이터를 표시하여야하는 상황을 접하게 되는데, 전체 레코드를 메모리에 불러들인 후 작업되는 이러한 형태는 불가능해진다. 

## admin-data-table의 조건
그래서 우리팀은 직접 관리자용 데이터테이블을 개발하기로 했다. 아래와 같은 조건들을 충족시키는 데이터테이블 라이브러리를 찾고 있다면 잘 찾아온 것이다.

- 표시되는 테이블의 **모든 컬럼은 검색 조건에 포함**될 수 있어야 한다.
- 여러 컬럼을 **AND 연산으로 복합 검색이 가능**하여야 한다.
- 또한 검색 조건시 값에 **<, >, <=, >=, % 등의 SQL 조건문을 사용**할 수 있어야 한다.
- 표시되는 모든 테이블의 컬럼은 정렬 조건에 포함될 수 있어야 한다.
- 현재의 정렬이나 검색 상태 등, 레코드 선택 상황에 대하여 즉각 인지할 수 있도록 시각적 힌트를 많이 사용한다.
- 데이터의 맥락을 시각적으로 보장하기 위하여 페이지 전환이 일어나지 않는 `more`형 페이징을 구현한다.
- 테이블에서 특정 레코드(행)가 삭제되거나 추가되어도 `more`에서 누락이나 중복이 발생하지 않도록 구현한다.
- 레코드별 작업이나 선택된 다중 레코드에 대한 작업이 가능하여야하며, **각 라우트(페이지)별로도 커스텀이 가능**하여야 한다.
- 테이블의 레코드들을 일괄 선택하여 작업하는 것이 가능하여야 한다.
- 기본적으로 데스크탑에 최적화시키며, 컬럼의 수와 상관없이 전체 데이터를 볼 수 있도록 고려되어야한다.
- 특정 조회 조건에 대해서 URL 접근이 가능하여야한다(공유 및 저장, 특정 데이터 조건으로 바로 접근)


# 프론트엔드 사용법
여기저기에 쓸 예정이므로 특정 라이브러리(React,Vue 등)에 종속성을 가지지 않게 jQuery만 붙들고 늘어지도록 하였지만... 그럼에도 불구하고 아래의 라이브러리들이 필요하긴 한다. 어차피 어드민 페이지에만 출력할 것을 고려하고 있으니 성능 신경쓰지 말고 다 갖다 붙이자.
```html
<head>
  <!-- Bootstrap CSS -->
  <link href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet" crossorigin="anonymous">
  
  <!-- jQuery JS -->
  <script src="//code.jquery.com/jquery-2.2.4.min.js"></script>
  
  <!-- Bootstrap JS -->
  <script src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
  
  <!-- jQuery UI JS -->
  <script src="//code.jquery.com/ui/1.12.0/jquery-ui.js"></script>

  <!-- AdminDataTable JS -->
  <script src="./admin-data-table.js"></script>
</head>
```

표시를 원하는 곳에 아래와 같은 마크업을 삽입한다.
```html
<div
	class="dataTable"
	data-url="/admin/images/rows"
	data-columns="id,_id,user_id,status,used,name,created_at,updated_at,ext,resource,thumbnail,width,height,brightness,hue"
	data-order-by="id desc"
	data-init-skip="y"
	data-batch-jobs="deleteAll:삭제"
	data-record-jobs="viewRow:보기,updateRow:수정,deleteRow:삭제"
></div>
```

실행은 아래와 같이 해주면 된다.
```javascript
<script>
  $('.dataTable').adminDataTable();
</script>
```

# 백엔드 사용법
표시할 데이터는 전적으로 서버에서 생성하여 반환을 하여야한다. 일단 `data-url`에 지정된 주소로 요청이 가능하도록 라우팅을 생성하고, 아래의 값들을 이용하여 조건식을 만들면 된다.
```php
// back-end php example source
$clauses = json_decode($request->get('clauses')); // 복합 조건이므로 json으로 들어온다. 파싱해서 써먹자.
$orderBy = $requset->get('orderBy'); // 'id desc'와 같이 스페이스로 구분되어 있다.
$skip = $request->get('skip'); // limit(20, $skip)과 같이 레코드를 몇개 스킵할 것인가 값이 들어있다.

// 어쩌구 저쩌구 쿼리수행
$total = $query::getCount("....");
$resultRows = $query::getRows("....");

$result = array( // status, total, rows(array) 들을 가지고 있는 배열을 만들고,
	'status' => 'done',
	'total' => $total,
	'rows' => $resultRows
);
response($result, 'json'); // json 형태로 응답해준다.
```
