## 1. RDS인스턴스 생성하기
* RDS(Relational Database Service) : AWS에서 지원하는 클라우드 기반 관계형 데이터베이스. 
1) RDS 대시보드에서 데이터베이스 생성을 클릭한다
2) 엔진 옵션에서는 mariaDB를 선택한다
    - 비용 : RDS는 라이센스 비용의 영향을 받는다. ORACLE, MSSQL이 오픈소스인 MySQL, MariaDB, PostgreSQL보다 동일 사양대비 가격이 높다
    - Amazon Aurora교체 용이성 : Amazon Aurora는 AWS에서 MySQL과 PostgreSQL을 클라우드 기반에 맞게 재구성한 DB이다
    <br> 하지만 Aurora는 프리티어대상이 아니며 월10만원 이상이기 때문에 MariaDB로 시작한 후 규모가 커지면 Aurora로 넘어가면 된다
    - MariaDB는 MySQL 기반으로 만들어졌다
3) 사용 버전을 선택
    * `MariaDB 10.2.21` 선택
4) 템플릿
    * 프리티어 선택
5) 인스턴스크기 
    * db.t2.micro -1 vCPU, 1GiB RAM
6) 스토리지
    * 스토리지유형 : 범용(SSD)
    * 할당된 스토리지 : 20
7) 설정
    * 인스턴스식별자/ 마스터 사용자이름/ 암호 를 설정한다
8) 연결
    * 퍼블릭 엑세스 가능 : '예'에 체크
        - 이후 보안그룹에서 지정된 IP만 접근 하도록 막을 것이다
9) 파라미터 그룹의 변경은 이후에 진행한다
10) 설정이 끝난 후 완료 버튼을 클릭하면 생성과정이 진행된다.

## 2. RDS 운영환경에 맞는 파라미터 설정하기

RDS를 처음 생성하면 다음 설정을 필수로 해야한다

1) 타임존
2) Character Set
3) Max Connection
    1. RDS/파라미터 그룹 선택 후 파라미터 그룹 생성을 클릭하여 생성한다(인스턴스와 버전을 맞춰준다)
    2. 생성한 그룹을 클릭하여 파라미터 편집으로 들어간다
    3. time_zone을 검색하여 Asia/Seoul 을 선택한다
    4. 기타 Charcter set을 변경한다
    ![파일_ㅎㅎ000](https://user-images.githubusercontent.com/58330668/112715721-5e161700-8f25-11eb-888a-42c1dcd3a5b8.jpeg)
    5. Max Connection 수정
        * Max Connection은 인스턴스 사양에따라 자동으로 정해진다
        * 현재 프리티어 사양으로는 약 60개의 커넥션만 가능하여 좀 더 넉넉한 값으로 지정한다(150)
        * 이후 RDS 사양을 높이게된다면 기본값으로 다시 돌려놓으면 된다
    6. 저장버튼을 클릭하여 설정을 마무리한다

4) 생성한 파라미터 그룹을 데이터베이스에 연결하기
    1. RDS/데이터베이스에서 인스턴스 선택 후 수정을 클릭한다
    2. 추가구성/데이터베이스 옵션에서 DB파라미터그룹을 방금 생성한 신규 파라미터그룹으로 변경한다
    3. 저장후 즉시적용을 누른다
        * 예약된 다음 유지 시간으로하면 새벽시간대에 진행된다. 수정사항이 반영되는동안 DB가 작동하지않을 수 있으므로 예약시간을 걸라는 의미지만
        오픈전 이기때문에 즉시적용한다

5) 로컬PC에서 RDS 접속하기
    1. 로컬PC에서 접근하기 위해 RDS의 보안 그룹에 1. 현재 내 PC의 IP, 2. EC2의 보안그룹 아이디를 인바운드로 추가한다
        - 인바운드 규칙 유형에서 MYSQL/Aurora를 선택하면 자동으로 3306포트가 선택된다
        - EC2의 경우 이후에 2대 3대가 될 수도 있는데, 매번 IP를 등록할 수는 없으니 보편적으로 이렇게 보안그룹 간에 연동을 진행한다
    2. RDS와 개인PC, EC2 간의 연동 설정을 모두 되었다!
    
6) Database 플러그인 설치
    1. 인텔리제이에 Database 플러그인을 설치한다. Database Navigator(* 최신버전에서는 서비스를 제공하고있지않아 인텔리제이 1.4이하 버전을 사용해야한다)
    2. 플러그인 설치 후 인텔리제이 재시작
    3. ctrl + Shift + a (Action 검색) 으로 Database Browser 검색 -> 왼쪽 사이드바에 DB Browser탭이 노출된다
    4. +버튼을 클릭 후 MYSQL을 클릭한다
    5. RDS 접속 정보를 등록. HOST에는 RDS 엔트포인트를 넣어주면된다
    6. ping test하고 apply 후 OK

7) charcter set 확인 후 변경
    1. character_set_database, collation_database 은 파라미터그룹으로 설정이 변경되지않는다. 따라서 alter문으로 변경
    ```
        use ASW RDS DB이름
        show variables like 'c%';
        alter database DB이름 
        charcter set = 'utf8mb4'
        collate = 'utf8mb4_general_ci';
    ```
8) time_zone 확인
    ```
        select @@time_zone, now();
    ```
9) test 테이블 만들어서 데이터 잘 들어가나 확인

10) RDS 설정 끝!
  
 
    
        
    

            
  
     
    