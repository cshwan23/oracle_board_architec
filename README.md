# oracle_board_architec
오라클 게시판 설계 
    
    <게시판 관련 테이블>

    create table board (
         b_no   number
         ,primary key (b_no)
    ------------------------
         , group_no
         , print_no
         , print_level

    );
    [ 모든 테이블에 PK(고유번호)가 있는 이유? ]

    > 행과 행이 서로 구별할 수 있는 방법.(pk를 주면 된다.)
    > pk번호가 없으면 게시판의 글을 나열 할 때 기준이 사라진다.

    [ group_no 그룹 번호가 있는 이유 ? ]

    > 메인들의 후손글들이 같이 뭉쳐다녀야하는데 첫번째 sort 기준이 된다.
    > sort 하기 위해서.

    [ print_no 프린트 번호가 있는 이유 ? ]

    > 같은 그룹 내에 누가 먼저 나올것인가?
    > 같은 그룹 내의 정렬 기준
    > 출력 순서 번호

    [ print_level 프린트 레벨이 있는 이유 ? ]

    > 들여쓰기 레벨 단계 (정렬과 관계x)
    > 누가 누구한테 댓글을 달았는지 


    [아래의 게시글의 그룹번호/프린트번호/프린트레벨은?]

    서울 시장은 누가 되나              3/0/0
        ㄴre:안철수                         3/1/1
        ㄴre:황보민                         3/2/1
            ㄴre:열혈지지                  3/3/2
                ㄴre:누가그래              3/4/3
        ㄴre:나경원                         3/5/1

    [처음 글을 올린다면?]

    insert into board (
    b_no, subject, wirter, content, pwd, email, group_no, print_no, print_level )
    value (
        (select nval(max(b_no),0)+1 from board)   --글번호
        ,(select nval(max(b_no),0)+1 from board)  -- 그룹번호
        ,0                                                                   -- 프린트번호
        ,0                                                                   -- 프린트레벨
    )
    print 번호 : 자기의 그룹번호를 그대로 갖다 쓰자

    [메인글(엄마) 조회수 1 증가시키기]

    update board set readcount = readcount +1 where b_no = 1;

    [엄마의 후손글들의 출력번호(print_no)를 1 증가시키기]

    조건 1. 엄마의 그룹번호와 동일하고 
    조건 2. 엄마의 출력순서번호 보다 큰 놈들

    update board set print_no = print_no + 1
    where group_no = (select group_no from board where b_no=1)
              group_no < (select print_no from board where b_no=1)


    [일련번호를 역순으로 넣기]

    총 테이블 행의 개수 - 일련번호 + 1

    select
         -- 총 테이블 행의 개수(6) - 일련번호 (1) + 1 = 6
         (select count(*) from board) - ROWNUM +1 "번호"
         , b . *
    from
    (
         select
              decode(print_level,1,' ㄴ',2,' ㄴ',3,' ㄴ','')||subject "제목"
              ,wirter "글쓴이"
              ,reg_date "등록일"
              ,readcount "조회수"
         from board
         order by group_no desc
                        ,print_no asc
    ) b;

    일련번호 정순 -> 역순으로 바꾸는 로직
    ( select count(*) from board ) - rownum +1 



    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.155 ***중요*** board 테이블 안의 데이터를 웹브라우저에 출력할 때 select 문?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    1. 일련번호 집어넣기 (1~6)

    : 인라인뷰 / rownum 



    2. 일련번호 역순으로 바꾸기 (6~1)

    : 테이블 행의 개수 - 일련번호 + 1 
    : (select count(*) from board) - ROWNUM + 1


    3. 들여쓰기 공백+ㄴ 집어넣기

    : lpad / decode
    lpad( ' ',print_level*5,' ')||decode(print_no,0,'','ㄴ')||subject



    [아래와 같은 테이블을 생성하기]

    --------------------------------------------------------------------
    이렇게 만들어야할까? 
    --------------------------------------------------------------------
    create table developer (
          -- 개발자 pk
          developer_no      number(3)      not null     
         -- 개발자 이름
         ,developer_name      varchar2(30)      not null
         -- 개발자 주민번호
         ,jumin_num      varchar2(30)     not null      unique
         -- 개발자 종교
         ,religion      varchar2(30)     not null
         -- 개발자 학력
         ,school      varchar2(30)     not null
         -- 개발자 기술
         ,skill      varchar2(30)     not null
         -- 개발자 졸업일
         ,graduate_date      date     not null
    )
    --------------------------------------------------------------------
    ★★★ 아니다. 이렇게 만들어야 한다. ★★★
    --------------------------------------------------------------------
    --------------------------------------------------------------------
    [ 1 ] 목록을 관리하는 테이블이 있어야 한다.(테이블 명 code_~)
    --------------------------------------------------------------------
    1. 스킬 목록
    create table code_skill(
          skill_code     number(3)
         ,skill_name     varchar2(20)     not null     unique
         ,primary key(skill_code)
    );
    insert into code_skill values(1, 'Java');
    insert into code_skill values(2, 'JSP');
    insert into code_skill values(3, 'ASP');
    insert into code_skill values(4, 'PHP');
    insert into code_skill values(5, 'Delphi');
    --------------------------------------------------------------------
    2. 종교 목록
    create table code_religion(
         religion_code     number(3)
         ,religion_name     varchar2(20)     not null     unique
         ,primary key(religion_code)
    );
    insert into code_religion values(1, '기독교');
    insert into code_religion values(2, '천주교');
    insert into code_religion values(3, '불교');
    insert into code_religion values(4, '이슬람교');
    insert into code_religion values(5, '무교');
    --------------------------------------------------------------------
    3. 학력 목록
    create table code_school(
         school_code     number(3)
         ,school_name     varchar2(20)     not null     unique
         ,primary key(school_code)
    );
    insert into code_school values(1, '고졸');
    insert into code_school values(2, '전문대졸');
    insert into code_school values(3, '일반대졸');
    --------------------------------------------------------------------
    [ 2 ] 개발자를 관리하는 테이블이 있어야 한다.(code_~)
    --------------------------------------------------------------------
    create table developer (
          -- 개발자 번호 pk
          developer_no      number(3)      not null     
         -- 개발자 이름
         ,developer_name      varchar2(14)      not null
         -- 개발자 주민번호
         ,jumin_num      varchar2(28)     not null      unique
         -- 개발자 종교 FK
         ,religion_code      number(3)     not null
         -- 개발자 학력 FK
         ,school_code      number(3)     not null
         -- 개발자 기술( 스킬은 여러개 선택 가능하니 따로 테이블을 만든다. )
         ,skillvarchar2(30)not null 
         -- 개발자 졸업일
         ,graduate_date      date     not null

    ,primary key (developer_no)
    ,foreign key (school_code) references code_school(school_code)
    ,foreign key (religion_code) references code_religion(religion_code)
    )
    --------------------------------------------------------------------
    [ 3 ] 개발자가 선택한 기술을 저장하는 developer 테이블 생성
    --------------------------------------------------------------------
    create table code_skill (
         -- 개발자 번호
         developer_no number(3) not null
         -- 스킬 번호
         ,skill_code number(3) not null
         -- FK developer_no(개발자번호)는 developer 테이블의 developer_no를 참조한다.
         ,foreign key(developer_no) references developer(developer_no)
         -- FK skill_code(스킬번호)는 code skill 테이블의 skill_code(스킬번호)를 참조한다.
         ,foreign key (skill_code) references code_skill(skill_code)
    );
    select * from developer_skill;
    --------------------------------------------------------------------
    --목록을 따로 관리하는 이유?
    --------------------------------------------------------------------
    -- 목록의 번호를 갖다 쓰면 데이터베이스 용량을 줄일 수 있다.
    -- 목록을 여러 페이지에서 갖다 쓴다면 목록의 통일성을 이룰 수 있다.


    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.157 아래와 같은 데이터가 웹서버로 들어왔다.
    -- 설계된 테이블에 데이터를 입력하는 insert 구문은?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- 개발자 이름 =>문유진
    -- 개발자 주민번호 앞 =>011215
    -- 개발자 주민번호 뒤 =>401814
    -- 개발자 종교 =>무교
    -- 개발자 학력 =>일반대졸
    -- 개발자 스킬 =>java,jsp
    -- 개발자 졸업일 =>2019-12-25
    ----------------------------------------------------------------
    -- 개발자 정보 데이터 입력
    ----------------------------------------------------------------
    insert into developer (
         developer_no
         ,developer_name
         ,jumin_num
         ,religion_code
         ,school_code
         ,graduate_date
    )values (
         (select nvl ( max (developer_no) , 0 ) + 1 from developer)
         ,'문유진'
         ,'011215-4018144'
         ,(select religion_code from code_religion where religion_name='무교')
         ,(select school_code from code_school where school_name='일반대졸')
         ,to_date('2019-12-25','YYYY-MM-DD')
    );
    select *from developer;
    select *from developer_skill;
    ----------------------------------------------------------------
    -- 스킬 관련 개발자 데이터 입력
    ----------------------------------------------------------------
    insert into developer_skill(
         developer_no
         ,skill_code
    )values(
         (select max(developer_no) from developer)
         ,(select skill_code from code_skill where skill_name='Java')
    );
    ----------------------------------------------------------------
    insert into developer_skill(
         developer_no
         ,skill_code
    )values(
         (select max(developer_no) from developer)
         ,(select skill_code from code_skill where skill_name='JSP')
    );
    ----------------------------------------------------------------
    -- 위 developer_skill insert 코딩을 병합
    ----------------------------------------------------------------

    insert into developer_skill(
         developer_no
         ,skill_code
    )
    select (select max(developer_no) from developer), skill_code from code_skill where skill_name='Java'
    union
    select (select max(developer_no) from developer), skill_code from code_skill where skill_name='JSP'
    ;
    ----------------------------------------------------------------
