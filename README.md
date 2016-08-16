# admin-data-table
어드민 관련 페이지를 만들려면 꼭 필요한 것이 두 가지 있다. 바로 데이터를 시각화한 차트와 실제 데이터를 보여주는 데이터테이블이다. 테이터테이블은 말 그대로 `table` 태그를 이용하면 되므로 쌩으로 써도 되고, 유명한 https://datatables.net 을 써도 무방하지만 아래와 같은 단점이 있다.

## HTML Table의 단점
기본적인 테이블을 쓰는 경우에는 열람 이외에 보다 더 자세한 작업이 불가능하다. 한마디로 정적인 인터페이스다. 물론 구석에 수정 및 삭제 링크를 두어 추가 작업을 하도록 여지를 둘 순 있지만, 무엇보다 원하는 데이터로의 접근이 썩 편하지가 못하다. 이를테면 각 컬럼별 정렬 조건을 추가하거나 특정 컬럼의 특정값으로 검색하는게 여의치 않다(보통의 경우에 테이블 외에 검색 폼을 두어 조건과 값을 입력하도록 되어 있지만 직관성이 부족하고 AND 연산폼을 구현하는게 쉽지 않다).

## DataTable.js의 단점
일단 이 라이브러리는 굉장히 편하고 빠르고 좋다. 보통의 경우에는 이 라이브러리를 통해서 간단하게 해치울 수 있을 것이다. 하지만 관리자단에서 특정 테이블을 관리한다는 것은 몇만건~몇억건의 데이터를 표시하여야하는 상황을 접하게 되는데, 전체 레코드를 메모리에 불러들인 후 작업되는 이러한 형태는 불가능해진다. 

## admin-data-table의 조건
그래서 우리팀은 직접 관리자용 데이터테이블을 개발하기로 한다. 이게 무조건 좋다고 할 수는 없지만 적어도 아래와 같은 우리팀의 목적에는 부합하도록 만들 수 있다.

- 표시되는 모든 테이블의 컬럼은 검색 조건에 포함될 수 있어야 한다.
- 여러 테이블 컬럼을 AND 연산으로 검색 조건에 포함할 수 있어야 한다.
- 또한 검색 조건시 값에 <, >, <=, >=, % 등의 SQL 조건문을 사용할 수 있어야 한다.
- 표시되는 모든 테이블의 컬럼은 정렬 조건에 포함될 수 있어야 한다.
- 데이터의 맥락을 시각적으로 보장하기 위하여 페이지 전환이 일어나지 않는 `more`형 페이징을 구현한다.
- 테이블에서 특정 레코드(행)가 삭제되거나 추가되어도 `more`에서 누락이나 중복이 발생하지 않도록 구현한다.
- 각 라우트(페이지)별로 레코드별 작업이나 선택된 레코드에 대한 작업을 커스텀할 수 있어야 한다.
- 테이블의 레코드들을 일괄 선택하여 작업하는 것이 가능하여야 한다.
- 이 외에 정렬이나 검색 등, 레코드 선택 상황에 대하여 즉각 인지할 수 있도록 시각적 힌트를 많이 사용한다.
- 