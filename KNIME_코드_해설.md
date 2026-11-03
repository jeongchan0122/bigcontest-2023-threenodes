# KNIME 코드 해설: 데이터 전처리

> 대회 결과보고서에 실었던 KNIME 워크플로 해설을 옮긴 것입니다. `전처리EDA_데이터제외.knwf`의 노드들이 무엇을 하는지 순서대로 설명합니다. 문장은 원문 그대로 두었습니다.
>
> 숫자(행 수)는 원본 데이터 기준이며, 데이터는 저장소에 포함되어 있지 않습니다.

## 목차

1. 밀린 데이터 처리 (empty_Y, empty_Z, empty_AA)
2. 공연 정보 결측치 채우기
3. 장르 결측치 채우기
4. 멤버십 구분
5. 좌석 특성 구분
6. 추가 결측치 처리 (예매 시작일, 장르)
7. 데이터 형태 변환

---

## 1) 밀린 데이터 처리 – empty_Y, empty_Z, empty_AA 컬럼 처리
데이터를 확인한 결과, member_yn 컬럼에 있어야할 값이 밀려서 empty_Y, empty_Z, empty_AA 컬럼으로 해당값이 채워져있는 경우들이 있음. 따라서 empty_Y,empty_Z,empty_AA컬럼의 값은 Y아니면 N의 값을 가짐. member_yn 칼럼 뿐만 아니라 empty칼럼이 공백이 아닌 행은 다른 컬럼에 있어야할 값이 다음 칼럼으로 밀려있는 행들이 존재하는 것을 확인함. 따라서 이러한 이상치가 존재할 경우 데이터 분석에 해를 가할 것으로 판단이 되어 밀린 데이터들을 처리하고자 함
&lt;처리 방법>
(a) Row splitter노드를 이용해 empty_AA가 결측치인 행과 아닌 행을 일차적으로 분리함. 이는 empty_AA가 결측치가 아닌 행을 처리하기 위함임.
(b) Row splitter노드를 이용해 empty_AA가 결측치인 행들 중에서 empty_Y가 결측치인 행과 아닌 행으로 분리함. 이는 empty_Y가 결측치가 아닌 행을 처리하기 위함임.
(c) Row splitter노드를 이용해 empty_Y가 결측치인 행들 중에서 empty_Z가 결측치인 행과 아닌 행으로 분리함. 이는 empty_Z가 결측치가 아닌 행을 처리하기 위함임.

**(a)과정**
empty_AA에 값이 존재하는 행들의 데이터(총 882개)의 정보 확인 : discount_type의 값들이 다음 칼럼인 performance_code칼럼으로 밀려있음. 그렇기 때문에 추가적으로 performance_code 값은 pre_open_date 값으로, pre_open_date값들은 open_date 칼럼으로 순차적으로 밀려있는 상태임. empty_AA에 값이 존재하는 행들의 discount_type칼럼의 고윳값은 두 가지로 첫번째는 '초'이고 그 개수는 816개이며, 두번째는 ‘학생할인(초)’는 66개임.

**(a)-1**
discount_type의 값이 ‘초’인 행들은 string manipulation 노드를 이용해 이 값을 공백으로 만들고 ‘학생할인(초)’ 인 행들과 concatenate노드를 이용해 합침. 그 후 string manipulation노드를 이용해 discount_type의 값을 performance_code칼럼의 값들과 합침. 이는 performance_code칼럼의 데이터 값을 discount_type의 값으로 끌어오는 행동임. 이로써, 밀린 discount_type칼럼의 값들은 정상처리됨. 하지만 performance_code칼럼의 값들부터는 여전히 밀려있는 상황임.

**(a)-2**
column name extracter노드를 이용해 컬럼들의 이름을 모두 추출하고 unpivot노드와 column filter노드를 바탕으로 performance_code부터 member_yn의 컬럼들의 이름을 추출함. 그 후 lag column노드를 이용해 행으로 입력되어있는 컬럼들의 이름값들을 한칸씩 미루고 row filter노드를 이용해 밀려서 나온 결측치를 처리함.

**(a)-3**
Column filter노드를 이용해 performance_code를 삭제하고 (a)-2의 결과와 column renamer노드를 이용하여 밀려있는 데이터들을 끌어옴. 그 후Sorter노드를 이용해 row_id를 기준으로 오름차순 정렬함

**(a)-4**
performance_code, running_time컬럼의 값을 number to string노드를 이용해 타입을 숫자열에서 문자열로 바꾸어줌. 그리고 string to number를 이용하여 open_date와 intermission을 문자열에서 숫자로 바꾸어줌.

**(a)-5**
concatenate노드를 이용해 empty_AA를 처리한 데이터셋 882개와 Empty_Y,Z,AA 모두 데이터가 존재하지않는 행 1045997개와 합쳐 총 1046879개 행의 데이터 셋을 가짐

**(b)과정**
empty_Y 값이 존재하는 행들의 데이터(총 1175개)의 정보 확인 : 동일하게 member_yn의 값들이 밀려서 empty_Y로 옴, 따라서 empty_Y는 Y아니면 N의 값만을 가짐. performance_code 칼럼부터 intermission칼럼까지 결측인 상태임

**(b)-1.**
empty_Y가 결측치가 아닌 행 1175개의 데이터에서 column filter노드를 이용해 member_yn 칼럼을 제외함.

**(b)-2.**
empty_Y가 결측치가 아닌 행 1175개의 데이터에서 column filter를 이용해 empty_Y칼럼만 추출 후, Empty_Y칼럼을 member_yn으로 이름을 수정

**(b)-3.**
1번과 2번의 데이터를 column appender 노드를 이용해 합침. 이로써 member_yn칼럼에 결측치가 존재했던 1175개의 행이 처리됨.

**(c)과정**
empty_Z 값이 존재하는 행들의 데이터(총 521개)의 정보 확인: 동일하게 memberyn의 값들이 밀려서 empty_Z로 옴, 따라서 empty_Z는 Y아니면 N의 값만을 가짐. performance_code부터 intermission까지 결측인 상태임

**(c)-1.**
empty_Z가 결측치가 아닌 행 1175개의 데이터에서 column filter노드를 이용해 member_yn 칼럼을 제외함.

**(c)-2.**
empty_Z가 결측치가 아닌 행 1175개의 데이터에서 column filter를 이용해 empty_Y칼럼만 추출 후, empty_Z칼럼을 member_yn으로 이름을 수정

**(c)-3.**
1번과 2번의 데이터를 column appender 노드를 이용해 합침. 이로써 member_yn칼럼에 결측치가 존재했던 521개의 행이 처리됨.

**(d)**
(b)-3의 데이터 1175개와 (c)-3의 데이터 521개를 concatenate노드를 이용해 합침.
단, 이 데이터 1696개는 performance_code부터 intermission까지 여전히 결측인 상태임.

## 2) 공연에 대한 정보(performance_code부터 intermission칼럼)에 대한 결측치 채우기
&lt;처리방법>
공연에 대한 정보가 비어있는 행들의 집합=(b)-3과 공연에 대한 정보가 비어있지 않은 행들의 집합(a)-5을 비교하여 날짜 시간, 가격대가 모두 같은 공연들을 뽑아서 그러한 공연들을 그룹화하여 performance_code의 unique_count가 하나인 행들을 식별하여 값을 채워넣었음.
(338 개의 데이터는 이 방식으로 결측값을 채울 수 없었으므로 제거하였음.)
(a) 공연에 대한 정보가 비어있는 행들의 집합인 (b)-3 총 1696개의 행을 가진 데이터에 row_id_missing열을 rowID노드를 통해 추가함. 그리고 그 값으로는 그 행의 ID로 갖게 함.
(b) 공연에 대한 정보가 비어있지 않은 행들의 집합인 (a)-5 총 1046879개의 행을 가진 데이터에 row_id_not_missing열을 rowID노드를 통해 추가함. 그리고 그 값으로는 그 행의 ID로 갖게 함.
(c) joiner노드를 이용하여 Play_date,play_st_time,price컬럼이 같은 행들끼리 추출하니 49164개의 행이 추출되었음.
(d)groupby노드를 이용하여 row_id_missing을 기준으로 그룹화하고 performance_code부터 intermission까지 공연에 대한 정보관련 컬럼의 고윳값 개수를 집계방법으로 함.
(e)performance_code의 고윳값이 1인 행들을 row splitter노드로 추출함. 그리고 column filter를 이용하여 그 행들의 row_id_missing열을 추출함.
(f) (c)의 데이터와 (e)의 데이터의 row_id_missing열 기준 교집합 데이터를 joiner노드를 통해 추출함. 그 결과, 44189개의 행이 추출됨
(g) groupby노드를 이용하여 row_id_missing을 기준으로 그룹화하고 performance_code부터 intermission까지 공연에 대한 정보관련 컬럼의 최빈값을 집계방법으로 함.

**(j)**
그 후 1696의 행과 1046879의 행에서 joiner노드를 이용하여 Play_date,play_st_time,price가 같은 행들끼리 연결을 하여 49164의 교집합이 만들어짐. (a) 그 후 row id missing을 기준으로 그룹화를 하고 performance_code부터 intermission까지 고윳값 개수로 집계연산을 수행하여 1481개의 데이터셋이 만들어졌다. 그리고 row splitter노드를 이용해 performance_code의 고윳값 개수가 1개인 행들만 추출하여 1358개가 남았다. 그 1358개의 데이터의 row_id_missing값만을 column filter로 추출하였고 (a)와 1358개의 데이터를 joiner노드를 이용해 row_id_missing이 같은 데이터를 교집합하여 얻어 44189개의 행 데이터를 얻었다.
공연에 대한 정보(performance_code부터 intermission)가 결측인 행들이 1696개 정도임.
이는 공연 날짜, 공연 시간, 가격 을 비교해서 3가지가 모두 같은 공연을 찾아서 참조하는 방식으로 값을 채워넣음. ex) 공연날짜가 1월 2일 공연 시간이 1900 가격이 10000 인 결측행a에 대해서
세가지 조건이 모두 같은 비결측행 b,c,d 가 있을 때, b,c,d 가 공연에 대한 정보가 모두 동일한 경우에는 결측항 a 의 공연에 대한 정보(결측값)을 해당 값으로 채워넣음
구체적으로는 각 결측행에 대해서 3가지 비교값이 모두 같은 비결측행이 대부분 여러개 식별되었는데, 그러한 비결측행들이 모두 공연코드, 장르, 예매시간 등이 동일한 경우에만 값을 채워넣는 방식을 사용하였습니다.

## 3) 장르 결측치 채우기-genre컬럼 처리
장르의 결측치를 채우는 방법으로 공연명 코드를 참조하였음. 예를 들어, 공연명 코드가 229인 데이터의 장르는 모두 교향곡이라면 공연명 코드가 229인데 데이터의 장르가 결측치일 때 교향곡으로 대체하는 방식임.
확인 결과, 공연명 코드가 2215인 행들의 장르는 모두 클래식, 229인 행들의 장르는 모두 교향곡이었음. 그리고 나머지는 이와 같은 참조 데이터가 없었음.
&lt;처리 방법>
(a) Row filter노드를 이용해 공연명코드가 2215, 229인 행들을 모두 제외함.
(b) Row filter노드를 이용해 공연명 코드가 2215인 행들 추출후 missing value 노드를 이용해 결측값을 클래식으로 바꿈
(c) Row filter노드를 이용해 공연명 코드가 229인 행들 추출 후 missing value 노드를 이용해 결측값을 교향곡으로 바꿈
(a),(b),(c)의 결과를 concatenate노드를 이용해 합침.
최종적으로 참고데이터에 맞게 채워서 장르 결측치 14092개 중 2715개는 결측값을 채우고 나머지 11377개는 아직 결측치를 가지고 있는 상태임.

## 4) 멤버십 구분
멤버십 칼럼들이 현재 membership_type_1\~6까지 그린, 블루, 싹틔우미 같은 멤버십이 명확한 순서나 규칙 없이 한 사람이 가지고 있는 멤버십이 나열되어 있는 형태임.
이 형태를 그 사람이 그린 멤버십을 가지고 있으면 1이고 가지고 있지 않으면 0인 '그린' 컬럼, 골드 멤버십을 가지고 있으면 1이고 가지고 있지 않으면 0인 '골드' 컬럼, 이런식으로 0,1로 구성된 여러 컬럼으로 재구성하려고 함.
멤버십 구성은 '골드', '그린', '블루', '무료', '노블', '싹틔우미' 6가지이므로 기존 membership_type1\~6 까지 6개의 컬럼을 '골드', '그린', '블루', '무료', '노블', '싹틔우미' 컬럼 6개로 대체하는 것임.
&lt;처리 방법>
(a) missing value노드를 이용하여 membership_type1부터 6까지 해당 멤버십을 보유하지않아 결측치인 값들을 모두 0으로 바꿈
(b) column combiner노드를 이용해 membership_type1\~6의 값들을 “,”로 합쳐서 새로운 combine string이라는 이름의 컬럼을 만들음
(c) rule-based row filter노드를 이용해 combine string에 골드가 있는지 여부에 따라 골드 비회원과 회원으로 분류함.
(d) rule engine노드를 이용해 골드 비회원들은 0으로, 골드 회원들은 1의 값을 가지도록 하는 새로운 골드라는 이름의 열을 추가함
(e) concatenate노드를 이용하여 골드 비회원인 데이터와 골드 회원인 데이터를 합치고, sorter노드를 이용해 row_id기준 오름차순으로 데이터를 정리함
이 과정을 골드 멤버십을 제외한 그린, 블루, 싹틔우미, 노블, 무료 멤버십에도 동일하게 적용을 하였고 column appender노드와 column filter노드를 이용해 기존 데이터에 '골드', '그린', '블루', '무료', '노블', '싹틔우미' 컬럼 6개를 추가한 형태의 결과물이 나옴

## 5) 좌석 특성 구분
좌석 특성 구분은 좌석(seat) 컬럼의 데이터를 &lt;층, 블록, 열, 좌석번호>로 분할하는 과정임. 이 과정에서 예술의 전당 좌석 배치도를 참고하였음.
&lt;처리 방법>
우선 cell splitter by position노드을 이용하여 seat컬럼에서 층에 대한 정보를 가진 ‘층’ 컬럼과 층을 제외한 값들로 이루어진 ‘그 외’ 컬럼을 새로 만들며 데이터값을 분할함.
좌석 같은 경우는 장소에 따라 크게 3가지 유형이 있었음. 이를 rule-based row filter노드를 이용해 각 유형으로 나눔

**유형 1) 층 다음에 BOX이 나오는 경우**

**유형 1-1) 열정보가 있는 경우(리사이틀홀)**
(a) String manipulation노드에서 strip함수를 이용해 앞뒤 공백을 삭제함
(b) Cell splitter by position 노드를 이용해 ‘그 외’ 컬럼에서 블록에 대한 정보를 가진 ‘블록’ 컬럼과 블록을 제외한 값들로 이루어진 ‘그외’ 컬럼을 새로 만들며 구분함. 그리고 column filter를 이용해 ‘그 외’ 컬럼을 삭제함
(c) String manipulation노드에서 indexof와 substr함수를 사용하여 ‘그외’ 컬럼에서 열 정보를 추출함. 그리고 strip함수를 이용해 앞뒤 공백을 삭제함.
(d) 동일한 방법으로 String manipulation노드에서 replace와 strip함수를 사용하여 ‘그외’ 컬럼에서 seat에 대한 좌석정보를 추출함. 그리고 column filter노드를 이용해 이제 불필요한 컬럼인 ‘그외’ 컬럼을 삭제함.

**유형 1-2) 유형으로는 열정보가 없는 경우(콘서트홀, IBK 홀)**
(a) String manipulation노드에서 strip함수를 이용해 앞뒤 공백을 삭제함
(b) Cell splitter노드를 이용해 블록과 ‘그 외’ 컬럼에서 블록에 대한 정보를 가진 ‘그 외_Arr[0]’ 컬럼과 블록을제외한 값들로 이루어진 ‘그 외_Arr[1]’ 컬럼을 새로 만들며 구분함
(c) 그리고 column filter노드로 블록 정보를 가지고 있는 ‘그 외 arr[0]’컬럼을 추출하고 string manipulation노드와 column filter노드를 이용하여 열 정보에 대한 공백 칼럼을 생성함. 이는 열이 없음을 보여주기 위함임.
(d) (b)와 (c)의 데이터를 column appender노드를 이용하여 합치고 column renamer노드를 이용해 ‘그 외_Arr[1]’컬럼의 이름을 좌석번호로 수정함.
(e) 정리된 데이터셋을 row splitter노드를 이용해 콘서트홀과 IBK홀로 분할하였음
(f) 콘서트홀 같은 경우는 row splitter노드로 2,3층을 분리함. 그 이유는 2층은 좌석이 2개씩 나열되고 3층은 좌석이 3개씩 나열되므로 열 부여 방식을 다르게 해야하기 때문임.
(g) column filter노드를 이용해 각각 좌석번호 컬럼을 추출 후 string to number노드를 이용하여 데이터타입을 정수로 바꿈. 그 후 math formula노드를 이용해 각각 2와 3으로 나누어 큰 정수로 올림하는 함수인 ceil을 이용해 해당 계산 값을 ‘열’컬럼의 값으로 갖도록 설정함. 다시 number to string노드를 통해 데이터타입을 문자열로 바꾸어주고 string manipulation노드를 이용하여 “열”이라는 글자를 숫자 뒤에 추가해주었음.
(h) 층별로 분리된 데이터를 column appender, concatenate노드를 이용하여 합침. 그리고 column filter노드를 이용하여 비어있었던 열 컬럼을 삭제하고 새롭게 부여한 열 컬럼을 column renamer를 통해 추가하였음.
(i) IBK홀 같은 경우의 2층 BOX의 좌석번호는 실질적으로 열번호에 가까워서 좌석번호에 열로 부여함
(j) (h)와 (i)의 데이터를 concatenate노드를 통해 합침.

**유형 2) 층 다음에 블록이 나오는 경우(콘서트홀,IBK홀)**
(a) String manipulation노드에서 strip함수를 이용해 앞뒤 공백을 삭제함
(b) Cell splitter by position 노드를 이용해 ‘그 외’ 컬럼에서 블록에 대한 정보를 가진 ‘블록’ 컬럼과 블록을 제외한 값들로 이루어진 ‘그외’ 컬럼을 새로 만들며 구분함. 그리고 column filter를 이용해 ‘그 외’ 컬럼을 삭제함
(c) String manipulation노드에서 indexof와 substr함수를 사용하여 ‘그외’ 컬럼에서 seat에 대한 열 정보를 추출함. 그리고 다시, String manipulation노드에서 strip함수를 이용해 앞뒤 공백을 없애고, ‘그외’ 컬럼에서 ‘열’ 컬럼의 값들을 뺀 좌석번호를 추출하는 ‘좌석번호’ 컬럼을 만들음. 그 후 column filter노드를 이용해 ‘그외’ 칼럼을 삭제함
(d) 리사이틀홀은 좌석 분류가 블록으로 되지 않는데, 블록으로 표시된 행이 존재함. 해당 데이터와 공연코드가 같은 행들을 참고하여 공연장 정보를 수정하려고 하였지만 공연코드가 같은 다른 행이 없기 때문에 이상치로 판단하여 row filter를 이용하여 제거하였음

**유형 3) BOX나 블록이 없는 경우(리사이틀홀)**

**유형 3-1) 좌석이 좌측 혹은 우측으로 분류된 경우**
홈페이지 좌석 배치도를 확인한 결과, 1층 좌측은 BOX1, 1층 우측은 BOX2였고 2층 좌측은 BOX3, 2층 우측은 BOX4임을 확인하였음. 따라서, 해당 값으로 대체하려함.
(a) row splitter노드를 통해 1층과 2층을 구분함.
(b) Cell splitter by position 노드를 이용해 ‘그 외’ 컬럼에서 블록에 대한 정보를 가진 ‘블록’ 컬럼과 블록을 제외한 값들로 이루어진 ‘그외’ 컬럼을 새로 만들며 구분함.
(c) string replacer노드를 이용해 1층의 우측은 BOX2로 좌측은 BOX1으로 치환함.
(d) 치환한 1층, 2층 데이터를 합치고 String manipulation노드에서 indexof와 substr함수를 사용하여 ‘그외’ 컬럼에서 seat에 대한 열 정보를 추출함. 그리고 다시, String manipulation노드에서 strip함수를 이용해 앞뒤 공백을 없애고, ‘그외’ 컬럼에서 ‘열’ 컬럼의 값들을 뺀 좌석번호를 추출하는 ‘좌석번호’ 컬럼을 만들음. 그 후 column filter노드를 이용해 ‘그외’ 칼럼을 삭제함

**유형 3-2) 좌측, 우측 데이터마저 없는 경우**
(a) String manipulation노드에서 indexof와 substr, replace함수를 사용하여 ‘그 외’ 컬럼에서 seat에 대한 열, 좌석정보를 추출함. 그리고 String manipulation노드에서 strip함수를 이용해 앞뒤 공백을 삭제하고 column filter노드를 이용하여 ‘그 외’ 컬럼을 삭제함.
(b) 홈페이지 좌석 배치도를 확인한 결과, BOX 정보가 없는 행들은 모두 중앙 메인좌석들이었음. 실제로 예술의 전당에서 중앙으로 분류하고 있지는 않지만, 리사이틀홀 내 다른 좌석과 구분하기 위해서, 중앙이라는 블록명 부여함. 이과정에서 column filter와 string manipulation노드를 이용함.
해당 5가지 유형의 데이터 셋을 모두 합치고sorter노드를 이용하여 row_id기준으로 오름차순으로 정렬함. 그리고 string manipulation노드를 이용하여 좌석 정보별로 띄어쓰기로 인해, 양 끝에 공백이 있을 수 있으므로 양쪽 공백을 제거해주었음
공백을 제외한 층, 블록, 열, 좌석번호에 대한 새 컬럼이 생겼고 이의 이름을 column renamer를 이용해 재설정해줌

## 6) 데이터 추가 결측치 처리
추가 전처리

## 6-1) open_date 결측치 처리
open_date가 공백인 공연 데이터가 존재함. 이는 전부 공연코드가 479인 공연들임. 공연들에 대하여 tran_date를 참고하여 결측치를 채웠음.
row splitter노드를 이용해 open_date가 공백인 행을 추출하고 missing value를 이용하여 open_date와 pre_open_date를 채움. 가장 빠른 거래일자인 20221129 값을 pre_open_date 값으로, 두 번째로 빠른 거래일자인 20221206 값을 open_date 값으로 채움.

## 6-2) genre 결측치 처리

3)에 이은 장르 결측치 처리 추가 시도, 아직 남은 장르 결측치 11377개가 존재함.
(a) intermission(3개), running_time(6개)의 value counter를 바탕으로 cross joiner노드를 이용하여 18개의
조합을 만들음.
(b) row filter노드를 이용하여 콘서트홀, IBK 챔버홀, 리사이틀홀을 분류함.
위 내용을 바탕으로 공연장별로 결측된 데이터의 running_time과 intermission의 조합에 대응되는 장르가 명시된 공연들에 대한 장르 최빈값을 참조하여 채울 것임. 예를 들어 콘서트홀에서 열린 공연시간이 120분 이고 쉬는 시간이 20 분인 행의 장르가 결측되어 있을 때, 콘서트홀에서 공연시간이 120분이고 쉬는 시간이 20분인 모든 공연에 대해서 장르의 최빈값을 구하여 해당값으로 결측값을 채움
&lt;콘서트홀 처리>
(a) groupby노드를 이용하여 intermission과 running_time을 기준으로 그룹핑하고 genre의 최빈값으로 집계함. 그리고 위 (a)와 joiner노드를 통해 조합들을 식별함. 이는 추정에 필요한 값을 선별하는 과정임.
(b) row splitter노드를 이용하여 장르 결측값이 있는 데이터와 없는 데이터를 분할함.
(c) rule engine노드를 이용하여 공연 시간, 쉬는 시간에 대한 장르 최빈값을 이용하여 장르 결측값 채움 (a)를 바탕으로 rule engine노드에서 running_time=100이고 intermission=20일 경우에는 ‘클래식’, running_time=95이고 intermission=20일 경우에는 ‘독주’로 채움.
(d) 남은 결측치는 missing value노드로 ‘최빈값’인 교향곡으로 채움.
&lt;IBK 챔버홀 처리>
(a) groupby노드를 이용하여 콘서트홀과 동일하게 intermission과 running_time을 기준으로 그룹핑하고 genre의 최빈값으로 집계함.
(b) 참조 데이터를 바탕으로 missing value 노드로 모두 ‘실내’로 채움
&lt;리사이틀홀 처리>
장르에 대한 결측치 없음.
각 장소마다 장르에 대한 결측치 처리 데이터를 concatenate노드를 통해 합침.

## 7) 모든 데이터 적절한 형태로 변환
(a) Running_time컬럼을 string to numbe노드를 이용하여 정수타입으로 변경함.
(b) Ticket_cancel컬럼은 명목형으로 표현되어야하므로 number to string노드를 이용하여 문자열 타입으로 변경함.
(c) 모든 date, time 관련 컬럼들을 number to string노드를 이용하여 yyyy:MMdd 형태로 변환함.
(d) Row splitter노드를 통해 tran_date의 길이가 1,2,3,4인 행들을 분류하고 string manipulation노드를 이용하여 00:00형태로 모두 바꿈. 그리고 concatenate노드로 분류해놨던 행들을 다시 합치고, sorter노드로 오름차순 정렬함
(e) Play_St_time같은 경우는 모두 4자리이기에 string manipulation노드를 이용하여 00:00형태로 바꿈
