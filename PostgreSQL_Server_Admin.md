# Part VI.  PostgreSQL 서버 관리

## 1. SW 설치 및 DB 생성  

✔︎ 이 문서는 Amazon EC2에 PostgreSQL 설치를 예시로 설명합니다.  
✔︎ [참고: How To Install PostgreSQL 12 on Amazon Linux 2](https://techviewleo.com/install-postgresql-12-on-amazon-linux/)  
✔︎ 시스템 최소 요구사항 확인
```
Amazon Linux 2 Virtual Machine  
2GB of RAM  
1 virtual cpu core  
1GB of disk space for installation  
User account with sudo or root user credentials  
```
✔︎ 설치 전 yum 패키지 최신으로 업데이트  

`$ sudo yum -y update`  
```
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
...  
280 packages excluded due to repository priority protections  
Resolving Dependencies  
--> Running transaction check  
---> Package … updated  
...
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================  
Package               Arch       Version         Repository        Size
========================================================================
Installing:
 ...
Updating:
 ...

Transaction Summary
========================================================================
Install ... Package
Upgrade ... Packages

Total download size: 283 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
...
------------------------------------------------------------------------
Total ...
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Updating   : ...
  Cleanup    : ...
  Verifying  : ...

Installed:
  ...

Updated:
  ...

Complete!
```

### 1.1. PostgreSQL Yum Repository 추가  

✔︎ amazon linux에서 설치 가능 버전 확인

`$ sudo amazon-linux-extras | grep postgresql`
```
  5  postgresql9.6            available    \
  6  postgresql10=latest      enabled      [ =10  =stable ]
 41  postgresql11             available    [ =11  =stable ]
```
✔︎ PostgreSQL v9.x는 "`sudo yum install -y postgresql`"로 설치가 되나, v10.x 이상은 Yum Repository를 먼저 설치하고 PostgreSQL을 설치해야 한다.   
✔︎ Amazon Linux 2 서버에 공식 PostgreSQL 리포지토리를 추가하려면 sudo 권한이 있는 루트 또는 사용자 계정으로 다음 명령을 실행합니다.

`$ sudo tee /etc/yum.repos.d/pgdg.repo<<EOF`  
`> [pgdg12]`  
`> name=PostgreSQL 12 for RHEL/CentOS 7 - x86_64`  
`> baseurl=https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-7-x86_64`  
`> enabled=1`  
`> gpgcheck=0`  
`> EOF`
```
[pgdg12]
name=PostgreSQL 12 for RHEL/CentOS 7 - x86_64
baseurl=https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-7-x86_64
enabled=1
gpgcheck=0
```
`$ ls -al /etc/yum.repos.d/pgdg.repo`
```
-rw-r--r--   1 root root  154 Jan 12 13:19 pgdg.repo
```
`$ sudo yum makecache`
```
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
...
pgdg12 ...
(1/24): ...
...
(24/24): ...
...
Metadata Cache Created
```

### 1.2. PostgreSQL 12 설치

✔︎ 리포지토리가 추가되면 다음 명령을 사용하여 Amazon Linux 2에 PostgreSQL 서버 및 클라이언트 패키지를 설치할 수 있습니다.

`$ sudo yum install postgresql12 postgresql12-server`
```
Loaded plugins: ...
281 packages excluded due to repository priority protections
Resolving Dependencies
--> Running transaction check
---> Package postgresql12.x86_64 0:12.5-1PGDG.rhel7 will be installed
--> Processing Dependency: postgresql12-libs(x86-64) = 12.5-1PGDG.rhel7 for package: postgresql12-12.5-1PGDG.rhel7.x86_64
---> Package postgresql12-server.x86_64 0:12.5-1PGDG.rhel7 will be installed
--> Running transaction check
---> Package postgresql12-libs.x86_64 0:12.5-1PGDG.rhel7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=================================================================================================================================
 Package                              Arch                    Version                              Repository               Size
=================================================================================================================================
Installing:
 postgresql12                         x86_64                  12.5-1PGDG.rhel7                     pgdg12                  1.6 M
 postgresql12-server                  x86_64                  12.5-1PGDG.rhel7                     pgdg12                  5.1 M
Installing for dependencies:
 postgresql12-libs                    x86_64                  12.5-1PGDG.rhel7                     pgdg12                  370 k

Transaction Summary
=================================================================================================================================
Install  2 Packages (+1 Dependent package)

Total download size: 7.0 M
Installed size: 30 M
Is this ok [y/d/N]: y


Downloading packages:
(1/3): postgresql12-libs-12.5-1PGDG.rhel7.x86_64.rpm ...
(2/3): postgresql12-12.5-1PGDG.rhel7.x86_64.rpm ...
(3/3): postgresql12-server-12.5-1PGDG.rhel7.x86_64.rpm ...
---------------------------------------------------------------------------------------------------------------------------------
Total                                                                                            1.0 MB/s | 7.0 MB  00:00:06
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql12-libs-12.5-1PGDG.rhel7.x86_64   1/3
  Installing : postgresql12-12.5-1PGDG.rhel7.x86_64        2/3
  Installing : postgresql12-server-12.5-1PGDG.rhel7.x86_64 3/3
  Verifying  : postgresql12-12.5-1PGDG.rhel7.x86_64        1/3
  Verifying  : postgresql12-server-12.5-1PGDG.rhel7.x86_64 2/3
  Verifying  : postgresql12-libs-12.5-1PGDG.rhel7.x86_64   3/3

Installed:
  postgresql12.x86_64 0:12.5-1PGDG.rhel7                      postgresql12-server.x86_64 0:12.5-1PGDG.rhel7

Dependency Installed:
  postgresql12-libs.x86_64 0:12.5-1PGDG.rhel7

Complete!
```

### 1.3. Database 초기화 및 기동

✔︎ 구성 파일 생성을 위해 데이터베이스 서버를 초기화해야 합니다. 이것은 설정 스크립트를 호출하여 수행니다.

`$ sudo /usr/pgsql-12/bin/postgresql-12-setup initdb`
```
Initializing database ... OK
```
✔︎ PostgreSQL 12 서버는 /var/lib/pgsql/12/data/postgresql.conf의 구성 파일을 사용합니다. 프로덕션 워크로드에 데이터베이스 서버를 사용하기 전에 모든 기본값을 검토하고 원하는대로 조정할 수 있습니다.

`$ sudo cat /var/lib/pgsql/12/data/postgresql.conf`
```
# -----------------------------
# PostgreSQL configuration file
# -----------------------------
#
# This file consists of lines of the form:
#
#   name = value
#

...
...
...

#--------------------------------------------------------------------------
# CUSTOMIZED OPTIONS
#--------------------------------------------------------------------------

# Add settings for extensions here
```
✔︎ OS 부팅시 서비스를 자동으로 시작하고 활성화하려면 systemctl enable 명령을 실행하십시오. 아래와 같이 부팅 시 자동 시작하는 서비스 정보가 생성되었다는 created symlink 메시지가 표시됩니다. systemctl enable 명령어에 --now 스위치를 사용하는 것은 자동 시작 설정을 하고 동시에 지금 바로 서비스도 시작(sudo systemctl start postgresql-12)하라는 의미입니다.

`$ sudo systemctl enable --now postgresql-12`
```
Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-12.service to /usr/lib/systemd/system/postgresql-12.service.
```
✔︎ 다음 명령어로 postgresql-12 서비스 실행 상태를 확인할 수 있습니다.  
`$ sudo systemctl status postgresql-12`
```
● postgresql-12.service - PostgreSQL 12 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-12.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2021-01-12 14:58:14 KST; 3min 31s ago
     Docs: https://www.postgresql.org/docs/12/static/
  Process: 11573 ExecStartPre=/usr/pgsql-12/bin/postgresql-12-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 11609 (postmaster)
    Tasks: 8
   Memory: 17.6M
   CGroup: /system.slice/postgresql-12.service
           ├─11609 /usr/pgsql-12/bin/postmaster -D /var/lib/pgsql/12/data/
           ├─11618 postgres: logger
           ├─11620 postgres: checkpointer
           ├─11621 postgres: background writer
           ├─11622 postgres: walwriter
           ├─11623 postgres: autovacuum launcher
           ├─11624 postgres: stats collector
           └─11625 postgres: logical replication launcher

Jan 12 14:58:14 bastion_host systemd[1]: Starting PostgreSQL 12 database server...
Jan 12 14:58:14 bastion_host postmaster[11609]: 2021-01-12 14:58:14.378 KST [11609] LOG:  starting PostgreSQL 12.5 on x8...64-bit
Jan 12 14:58:14 bastion_host postmaster[11609]: 2021-01-12 14:58:14.378 KST [11609] LOG:  listening on IPv4 address "127...t 5432
Jan 12 14:58:14 bastion_host postmaster[11609]: 2021-01-12 14:58:14.379 KST [11609] LOG:  listening on Unix socket "/var....5432"
Jan 12 14:58:14 bastion_host postmaster[11609]: 2021-01-12 14:58:14.382 KST [11609] LOG:  listening on Unix socket "/tmp....5432"
Jan 12 14:58:14 bastion_host postmaster[11609]: 2021-01-12 14:58:14.393 KST [11609] LOG:  redirecting log output to logg...rocess
Jan 12 14:58:14 bastion_host postmaster[11609]: 2021-01-12 14:58:14.393 KST [11609] HINT:  Future log output will appear..."log".
Jan 12 14:58:14 bastion_host systemd[1]: Started PostgreSQL 12 database server.
Hint: Some lines were ellipsized, use -l to show in full.
```

`$ ps -ef | grep postgres`
```
postgres 11609     1  0 14:58 ?        00:00:00 /usr/pgsql-12/bin/postmaster -D /var/lib/pgsql/12/data/
postgres 11618 11609  0 14:58 ?        00:00:00 postgres: logger
postgres 11620 11609  0 14:58 ?        00:00:00 postgres: checkpointer
postgres 11621 11609  0 14:58 ?        00:00:00 postgres: background writer
postgres 11622 11609  0 14:58 ?        00:00:00 postgres: walwriter
postgres 11623 11609  0 14:58 ?        00:00:00 postgres: autovacuum launcher
postgres 11624 11609  0 14:58 ?        00:00:00 postgres: stats collector
postgres 11625 11609  0 14:58 ?        00:00:00 postgres: logical replication launcher
```

### 1.4. PostgreSQL 관리 사용자 비밀번호 설정  

✔︎ DB 작업에 대한 권한을 에스컬레이션하는 데 사용할 PostgreSQL 관리자 비밀번호를 설정합니다.  

`$ sudo su - postgres`  
`$ psql -c "alter user postgres with password 'StrongPassword'"`
```
ALTER ROLE
```  

## 2. 서버 구성

### 2.1. CLI PATH 설정  

✔︎ DB 작업에 대한 CLI(Command Line Interface) 사용을 위해서는 CLI 실행 파일이 모여 있는 bin directory에 대한 PATH 설정이 필요하며, 설치 버전 및 OS에 따라 경로가 다를 수 있습니다. 현재 EC2 서버 예제에서는 “/usr/pgsql-12/bin” 경로를 추가해야 합니다.  

`# pg_ctl status`
```
bash: pg_ctl: command not found
```
`$ cd ~`
`$ pwd`
```
/var/lib/pgsql
```
`$ ls -al`
```
total 20
drwx------  3 postgres postgres   95 Jan 25 13:39 .
drwxr-xr-x 40 root     root     4096 Jan 12 14:19 ..
drwx------  4 postgres postgres   51 Jan 12 14:49 12
-rw-------  1 postgres postgres 1932 Jan 25 13:09 .bash_history
-rwx------  1 postgres postgres  318 Jan 25 13:39 .bash_profile
-rw-------  1 postgres postgres   34 Jan 25 13:47 .psql_history
-rw-------  1 postgres postgres 1044 Jan 25 13:35 .viminfo
```
`$ cat .bash_profile`
```
[ -f /etc/profile ] && source /etc/profile
PGDATA=/var/lib/pgsql/12/data
export PGDATA
# If you want to customize your settings,
# Use the file below. This is not overridden
# by the RPMS.
[ -f /var/lib/pgsql/.pgsql_profile ] && source /var/lib/pgsql/.pgsql_profile
# add PATH
PATH=$PATH:/usr/pgsql-12/bin
export PATH
```
`$ . ~/.bash_profile`
`$ echo $PATH`
```
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/usr/pgsql-12/bin
```
`$ pg_ctl status`
```
pg_ctl: server is running (PID: 10914)
/usr/pgsql-12/bin/postgres
```

### 2.2. 네트워크 설정  

✔︎ postgresql은 로컬 연결에 운영체제의 사용자 이름과 db의 사용자 이름이 일치하면 접근할 수 있는 피어(peer) 인증이 기본 값으로 설정되어 있습니다. 그래서 설치 후 별도의 비밀번호 요구 없이 DB 관리자 postgres 계정으로 기본 인증하여 psql이 실행되며 DB에 접근할 수 있습니다.

✔︎ 일반적으로 데이터베이스 서버는 네트워크를 통해 원격 접속하여 사용하며, 이를 위해 네트워크 원격 연결을 활성화하는 3가지 구성 설정이 필요합니다.  

#### 2.2.1. 네트워크 접근제어

✔︎ 원격에서 postgresql 접속 시도를 할 경우, 먼저 네트워크 방화벽에서 인바운드 접근이 허용되어야 합니다. 이 예제와 같이 EC2 서버의 경우 명시적으로 허용되어 있어야 합니다. postgreSQL 서버에 대한 Security Group의 inbound rule에 TCP 5432 포트 허용 규칙이 없다면 timed out 오류 현상이 발생합니다.  

* 이미지 파일 "postgresql_connection_timed_out_5432.jpg" 링크 예정  

✔︎ AWS EC2의 Security Group 목록에서 postgreSQL 설치 서버에 대한 Security Group을 선택하고 inbound rule에 TCP 5432 포트 허용 규칙을 추가합니다.  

* 이미지 파일 "postgresql_connection_add_port_5432.jpg" 링크 예정  

#### 2.2.2. DB서버 접근제어

✔︎ 원격 접속 요청이 방화벽을 통과하여 PostgreSQL DB에 접속 시도를 할 경우, DB 프로세스가 DB 작업 요청 허용 목록(DB listen address 목록)의 컴퓨터로 접속 시도를 하는지 체크합니다. DB listen address 목록 이외의 컴퓨터는 DB 접속 거부(Connection refused) 현상이 발생합니다.  

* 이미지 파일 "postgresql_connection_refuged_err.jpg" 링크 예정  

✔︎ postgresql 서버 구성 파일(/var/lib/pgsql/12/data/postgresql.conf)의 DB listen address 목록 파라미터(listen_addresses) 값에 접속을 시도하는 클라이언트를 추가합니다. 기본값은 listen_addresses = ’localhost’ 이며, 여기서는 편의상 모든 클라이언트를 허용하도록 listen_addresses = ’*’ 값으로 설정합니다. 변경 후에는 DB 재시작이 필요합니다.

`$ vi /var/lib/pgsql/12/data/postgresql.conf`
```
# -----------------------------
# PostgreSQL configuration file
# -----------------------------
...
...
...

#--------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#--------------------------------------------------------------------------

# - Connection Settings -

listen_addresses = '*'           # what IP address(es) to listen on;
                                 # comma-separated list of addresses;
                                 # defaults to 'localhost'; use '*' for all
                                 # (change requires restart)

...
...
...
```
`$ sudo systemctl stop postgresql-12`
`$ sudo systemctl start postgresql-12`

#### STEP 3: DB사용자 접근제어

✔︎ 원격 접속 요청을 시도하는 컴퓨터가 해당 IP나 IP Block으로 서버 접근을 허용 받았다면, 허용된 인증 방식으로 접속 시도를 하는지 체크합니다. 허용된 인증 방식 이외에는 DB 접속 오류 현상이 발생합니다.  

* 이미지 파일 "postgresql_connection_no_pg_hba_err.jpg" 링크 예정  

✔︎ postgresql 서버 인증 구성 파일(/var/lib/pgsql/12/data/pg_hba.conf)에서는 호스트 기반 인증(host-based authentication)으로 5가지 인증 요소에 대한 방식을 정의합니다. 일반적으로 “host all all 0.0.0.0/0 md5”를 추가하여 사용합니다. (상세 내용 매뉴얼 참조)

`$ vi /var/lib/pgsql/12/data/pg_hba.conf`
```
# PostgreSQL Client Authentication Configuration File
# ===================================================

# TYPE: 통신 암호화 방식을 정의하며, 보통 tcp/ip 연결을 위해 host로 설정한다.
# DATABASE: 접근 DB명을 정의하며, 보통 모든 DB로 all로 설정한다.
# USER: 접근 사용자명을 정의하며, 보통 모든 사용자로 all로 설정한다.
# ADDRESS: 클라이언트 주소를 cidr로 정의하며, 보통 모든 주소로 0.0.0.0/0로 설정한다.
# METHOD: DB 사용자인증 방식을 정의하며, 보통 비밀번호 방식으로 md5로 설정한다.

# TYPE  DATABASE        USER            ADDRESS                 METHOD 
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
host    all             all             0.0.0.0/0               md5
```
`$ sudo systemctl stop postgresql-12`  
`$ sudo systemctl start postgresql-12`

✔︎ postgresql DB의 카탈로그 테이블인 pg_user의 계정 정보를 참조해서 사용자 정보와 비밀번호가 일치하면 원격 사용자가 DB 서버에 접속됩니다.  

* 이미지 파일 "postgresql_connection_ok.jpg" 링크 예정  

## 3. 사용자와 역할

✔︎ [참고: PostgreSQL 사용자와 역할 관리](https://aws.amazon.com/ko/blogs/database/managing-postgresql-users-and-roles/)  

### 3.1. 사용자(User), 스키마(Schema), 역할(Role)  

✔︎ PostgreSQL 데이터베이스에서는 사용자와 스키마가 별도로 생성됩니다. 하지만 오라클 데이터베이스에서 사용자는 스키마와 동일합니다.

✔︎ PostgreSQL 데이터베이스에서 사용자는 역할에 로그인 권한이 부여된 것을 의미합니다.  

![User and Roles](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2019/03/01/managing-postgresql-users-2.gif)  

`CREATE USER myuser WITH PASSWORD 'secret_passwd';`  
`CREATE ROLE myuser WITH LOGIN PASSWORD 'secret_passwd';`
  
이 두 문장 모두 정확히 동일한 사용자를 만듭니다.  

✔︎ Oracle과 같은 다른 관계형 데이터베이스 관리 시스템 (RDBMS)에서 사용자와 역할은 서로 다른 두 엔티티입니다. Oracle에서는 데이터베이스에 로그인하는 데 역할을 사용할 수 없습니다. 이 역할은 권한 부여 및 기타 역할을 그룹화하는 용도로만 사용됩니다. 그런 다음 이 역할을 한 명 이상의 사용자에게 할당하여 모든 권한을 부여 할 수 있습니다.  

### 3.2 Public 스키마와 Public 역할  

✔︎ 기본적으로 새 데이터베이스가 생성될 때 public 스키마와 이 스키마에 대한 접근 권한이 부여된 public 역할이 생성됩니다.  
✔︎ 기본적으로 새 사용자와 역할이 생성될 때 public 역할이 부여됩니다. 이에 따라 명시적인 스키마를 지정하지 않는다면 새로운 사용자가 생성한 테이블은 일반적으로(search_path 환경 변수 설정에 따라) public 스키마에 생성됩니다.  
✔︎ 데이터베이스 읽기 전용 사용자를 생성할 경우에는 public 역할에서 부여한 권한이 회수되어야 합니다.  

* public 역할에서 public 스키마의 생성 권한 취소  
`REVOKE CREATE ON SCHEMA public FROM PUBLIC ;`  
* public 역할에서 mydatabase의 모든 권한 취소  
`REVOKE ALL ON DATABASE mydatabase FROM PUBLIC ;`  
 (mydatabase의 모든 권한이 public 역할에서 취소되어 모든 사용자가 명시적으로 허용된 권한만 사용 가능. 즉, mydatabase에 접속 조차 안됨)  

### 3.3 search_path 설정  

`"search_path"`는 데이터베이스에서 명시적인 스키마 지정 없이 단순 이름으로 객체(테이블 또는 함수 등)를 참조할 때 스키마가 검색되는 순서를 지정하는 환경 변수입니다.  
  
형식) "$user", public[, schema_names]  
예시)  

```
postgres=# show search_path;  
   search_path   
-----------------
 "$user", public
(1 row)
```  

"$user"는 현재 로그인 한 사용자의 이름(session_user)이며 별도로 사용자와 동일한 이름의 스키마를 생성하지 않는 한 일반적으로 같은 스키마는 없습니다. 다른 스키마에 동일한 이름의 개체가 존재할 경우에는 검색 경로에서 먼저 발견되는 객체가 사용됩니다.  

✔︎ search_path 값 설정  
  
* 모든 사용자에게 영구적인 search_path 값 지정을 위해서는 postgresql.conf 데이터베이스 구성 파일에 search_path=... 반영  
* 특정 사용자에게 영구적인 search_path 값 지정을 위해서는 “alter user 사용자명 set search_path=...;” 명령어 수행  
* 세션에서 임시적인 search_path 값 지정을 위해서는 set search_path=...; 명령어 수행  

### 3.4 데이터베이스 역할 생성  
  
![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2019/03/01/managing-postgresql-users-3.gif)  

데이터베이스, 스키마 및 스키마 개체 수준에서 권한을 부여해야 함. 예를 들어, 테이블에 대한 액세스 권한을 부여해야하는 경우 역할에 테이블이 있는 데이터베이스 및 스키마에 대한 액세스 권한이 있는지도 확인해야 함. 권한이 누락된 경우 역할은 테이블에 액세스 할 수 없음.  

### 3.5 사용자와 역할을 통한 액세스 제어 권장 방식  

![](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2019/03/01/managing-postgresql-users-1.gif)  

* master 사용자로 응용 프로그램 또는 사용 사례별로 역할 생성. 예) readonly, readwrite  
* 역할에 다양한 데이터베이스 개체에 액세스 할 수 있는 권한을 추가. 예) readonly역할은 SELECT 권한만 부여  
* 역할에는 필요한 최소한의 권한을 부여  
* 응용 프로그램 또는 고유한 기능에 따라 사용자 생성. 예) app_user, reporting_user.  
* 사용자에게 해당 권한을 보유한 역할을 할당하여 권한을 빠르게 부여. 예) readwrite역할을 app_user부여하고, readonly역할을 reporting_user에게 부여  
* 사용자에게 역할을 제거하여 언제든지 권한 회수 가능

### 3.6 사용자와 역할 생성 시나리오 예제  

✔︎ 데이터베이스 생성  

postgres=> `\l`  

```
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 rdsadmin  | rdsadmin | UTF8     | en_US.UTF-8 | en_US.UTF-8 | rdsadmin=CTc/rdsadmin
 template0 | rdsadmin | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/rdsadmin          +
           |          |          |             |             | rdsadmin=CTc/rdsadmin
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)
```
postgres=> `create database locdb with encoding 'UTF8' template template0 lc_collate 'C' lc_ctype 'ko_KR.UTF-8';`  
```
CREATE DATABASE
```  

postgres=> `\l`  
```
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 locdb     | postgres | UTF8     | C           | ko_KR.UTF-8 |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 rdsadmin  | rdsadmin | UTF8     | en_US.UTF-8 | en_US.UTF-8 | rdsadmin=CTc/rdsadmin
 template0 | rdsadmin | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/rdsadmin          +
           |          |          |             |             | rdsadmin=CTc/rdsadmin
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(5 rows)
```  

locdb=> `\d`  

```  
Did not find any relations.
```  

✔︎ 스키마 생성  

locdb=> `create schema locma;`

```
CREATE SCHEMA
```  

locdb=> `show search_path;`  

```
   search_path
-----------------
 "$user", public
(1 row)
```  

locdb=> `set search_path="$user", public, locma;`  
```
SET
```
locdb=> `show search_path;`  
```
      search_path
------------------------
 "$user", public, locma
(1 row)
```

✔︎ public 역할의 기본 권한 철회


locdb=> `revoke create on schema locma from public;`
```
REVOKE
```
locdb=> `revoke all on database locdb from public;`
```
REVOKE
```

✔︎ 기능(readonly, readwrite) 역할 생성 및 권한 부여

locdb=> `create role loc_readonly;`
```
CREATE ROLE
```

locdb=> `grant connect on database locdb to loc_readonly;`
```
GRANT
```
locdb=> `grant usage on schema locma to loc_readonly;`
```
GRANT
```
locdb=> `grant select on all tables in schema locma to loc_readonly;`
```
GRANT
```
locdb=> `alter default privileges in schema locma grant select on tables to loc_readonly;`
```
ALTER DEFAULT PRIVILEGES
```
locdb=> `create role loc_readwrite;`
```
CREATE ROLE
```
locdb=> `grant connect on database locdb to loc_readwrite;`
```
GRANT
```
locdb=> `grant usage, create on schema locma to loc_readwrite;`
```
GRANT
```
locdb=> `grant select, insert, update, delete on all tables in schema locma to loc_readwrite;`
```
GRANT
```
locdb=> `alter default privileges in schema locma grant select, insert, update, delete on tables to loc_readwrite;`
```
ALTER DEFAULT PRIVILEGES
```
locdb=> `grant usage on all sequences in schema locma to loc_readwrite;`
```
GRANT
```
locdb=> `alter default privileges in schema locma grant usage on sequences to loc_readwrite;`
```
ALTER DEFAULT PRIVILEGES
```

✔︎ 사용자 생성 및 역할 부여

locdb=> `create user reporting_user1 with password '...';`
```
CREATE ROLE
```
locdb=> `create user reporting_user2 with password '...';`
```
CREATE ROLE
```
locdb=> `create user app_user1 with password '...';`
```
CREATE ROLE
```
locdb=> `create user app_user2 with password '...';`
```
CREATE ROLE
```
locdb=> `grant loc_readonly to reporting_user1 ;`
```
GRANT ROLE
```
locdb=> `grant loc_readonly to reporting_user2 ;`
```
GRANT ROLE
```
locdb=> `grant loc_readwrite to app_user1 ;`
```
GRANT ROLE
```
locdb=> `grant loc_readwrite to app_user2 ;`
```
GRANT ROLE
```

✔︎ 데이터베이스 사용자 및 역할 조회 예제 (AWS RDS 예시)

```
SELECT 
      r.rolname, 
      ARRAY(SELECT b.rolname
            FROM pg_catalog.pg_auth_members m
            JOIN pg_catalog.pg_roles b ON (m.roleid = b.oid)
            WHERE m.member = r.oid) as memberof
FROM pg_catalog.pg_roles r
WHERE r.rolname NOT IN ('pg_signal_backend','rds_iam',
                        'rds_replication','rds_superuser',
                        'rdsadmin','rdsrepladmin')
ORDER BY 1;
```


## 4. 백업 및 복구 고려 사항  

### 4.1. dump 백업 및 복구 (pg_dump)  

✔︎ dump 백업 방법은 SQL을 이용하여 파일을 생성하는 것이며, 덤프 수행 시점의 동일한 데이터베이스로 복구가 가능합니다. PostgreSQL에서는 PG_DUMP 유틸리티를 지원하며 특정 데이터베이스를 선택하여 백업 및 복구 할 수 있다는 장점이 있습니다. 백업하려는 테이블의 읽기 액세스 권한이 있어야 가능하며, 만약 데이터베이스 전체를 백업하는 경우에는 슈퍼유저(postgres)로 사용해야 합니다.  

✔︎ pg_dump는 개별 DB를 백업하고 pg_dumpall은 전체 DB를 plain text 방식으로 백업합니다. plain text는 압축 백업 방식을 사용하는 것도 도움이 됩니다. (예: pg_dumpall | gzip > dumpall.sql.gz)  

✔︎ 복원 시에는 기본 포맷 형태인 plain text 백업의 경우에는 psql을 사용하고, custom 방식(-Fc) 백업의 경우에는 PG_RESTORE를 사용합니다. 자세한 문법은 명령어 --help 옵션으로 확인할 수 있습니다.  

✔︎ crontab을 이용하여 자동 백업을 수행할 경우에는 “.pgpass” 파일에 접속 정보를 기록하여 수행합니다.  

$ `cat .pgpass`  
```
hostname:portno:dbname:username:password
```
$ `crontab -l`
```  
## AWS RDS Daily Backup Time 07:30
30 07 * * 1-5 backup_aurora_postgres.sh >> postgres.log 2>&1
30 07 * * 1-5 backup_aurora_locdb.sh >> backup_rds_dump/locdb.log 2>&1
30 07 * * 1-5 backup_aurora_appdb.sh >> appdb.log 2>&1
```  
$ `cat backup_aurora_appdb.sh`
```
#!/bin/bash
BACKUP_DIR=/home/postgres/backup_rds_dump
DB_NAME=appdb
DATE=$(date +%Y%m%d)

find /home/postgres/backup_rds_dump -name 'appdb.*.dump' -mtime +8 -exec rm {} \; -print > /dev/null

# backup command (plain text):
pg_dump -h icheckdb.cluster-ct1dr3c2qztg.ap-northeast-2.rds.amazonaws.com -d appdb -U postgres -f /home/postgres/backup_rds_dump/appdb.$(date +%Y%m%d).dump
# restore command: psql -f appdb.$(date +%Y%m%d).dump -h localhost -d appdb -U postgres

# backup command (custom format):
pg_dump -h icheckdb.cluster-ct1dr3c2qztg.ap-northeast-2.rds.amazonaws.com -d appdb -U postgres -Fc -f /home/postgres/backup_rds_dump/appdb.$(date +%Y%m%d).Fc.dump
# restore command: pg_restore -v -h localhost -U postgres -d appdb appdb.$(date +%Y%m%d).Fc.dump
```  
$ `ls -ltr`  
```
-rw-r--r--. 1 postgres postgres   10073125  1월 25 07:30 appdb.20210125.dump
-rw-r--r--. 1 postgres postgres    1826059  1월 25 07:30 appdb.20210125.Fc.dump
```  

### 4.2. 오프라인 전체 백업 (Cold Backup).

✔︎ PostgreSQL은 DATA 디렉토리에 모든 데이터를 저장하고 관리합니다. 용량이 많지 않는 데이터들은 SQL DUMP로 받아 손쉽게 백업받아 복구할 수 있지만, 대용량 데이터인 경우에는 SQL DUMP보다 파일 시스템을 통째로 백업 받는 것이 복구시에 속도가 더 빠를 수도 있습니다. 하지만, 제약사항이 존재합니다. 첫째, 반드시 데이터베이스가 종료 된 상태에서 백업을 수행해야 하며, 둘째, 특정 데이터베이스만 백업되지 않고, 전체 백업을 해야 합니다. 마지막으로 리눅스-윈도우와 같이 이기종 호환은 불가능 합니다.

$ `echo $PGDATA`
```
/var/lib/pgsql/12/data
```
$ `echo $PGDATA`
```
/var/lib/pgsql/12/data
```
$ `pg_ctl -D $PGDATA -mf stop`
```
waiting for server to shut down.... done
server stopped
```
$ `cd /var/lib/pgsql/12`  
$ `ls -al`
```
total 4704
drwx------  4 postgres postgres      67 Jan 25 17:57 .
drwx------  3 postgres postgres      95 Jan 25 13:39 ..
drwx------  2 postgres postgres       6 Nov 12 05:34 backups
drwx------ 20 postgres postgres    4096 Jan 25 13:40 data
-rw-------  1 postgres postgres     918 Jan 12 14:49 initdb.log
```
$ `tar -zcvf data.tar data/`  
$ `ls -al`
```
total 4704
drwx------  4 postgres postgres      67 Jan 25 17:57 .
drwx------  3 postgres postgres      95 Jan 25 13:39 ..
drwx------  2 postgres postgres       6 Nov 12 05:34 backups
drwx------ 20 postgres postgres    4096 Jan 25 13:40 data
-rw-r--r--  1 postgres postgres 4807145 Jan 25 17:57 data.tar
-rw-------  1 postgres postgres     918 Jan 12 14:49 initdb.log
```
$ `rm -rf data/`  
$ `pg_ctl -D $PGDATA -mf start`
```
pg_ctl: directory "/var/lib/pgsql/12/data" does not exist
```
$ `tar -xvf data.tar`  
$ `pg_ctl -D $PGDATA -mf start`  
```
waiting for server to start....2021-01-25 18:12:16.045 KST [28538] LOG:  starting PostgreSQL 12.5 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-39), 64-bit
2021-01-25 18:12:16.045 KST [28538] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2021-01-25 18:12:16.045 KST [28538] LOG:  listening on IPv6 address "::", port 5432
2021-01-25 18:12:16.046 KST [28538] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2021-01-25 18:12:16.049 KST [28538] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2021-01-25 18:12:16.060 KST [28538] LOG:  redirecting log output to logging collector process
2021-01-25 18:12:16.060 KST [28538] HINT:  Future log output will appear in directory "log".
 done
server started
```

### 4.3. 장애 시점 복구 (PITR: Point In Time Recovery)  

✔︎ 데이터 파일과 트랜잭션 이력을 기록한 WAL(Write-Ahead Logging) 파일을 백업하고 보관하여 장애 시점 복구를 실현할 수 있습니다.  

#### 4.3.1. archive_mode 활성화  

✔︎ 장애 시점 복구를 위해서는 online 백업 모드를 활성화하기 위한 환경 변수 archvie_mode 값을 "on"으로 변경하고 Instance를 재시작하여 적용합니다.  

$ `psql`
```
psql (12.5)
Type "help" for help.
```
postgres=# `\d pg_settings`
```
               View "pg_catalog.pg_settings"
     Column      |  Type   | Collation | Nullable | Default
-----------------+---------+-----------+----------+---------
 name            | text    |           |          |
 setting         | text    |           |          |
 unit            | text    |           |          |
 category        | text    |           |          |
 short_desc      | text    |           |          |
 extra_desc      | text    |           |          |
 context         | text    |           |          |
 vartype         | text    |           |          |
 source          | text    |           |          |
 min_val         | text    |           |          |
 max_val         | text    |           |          |
 enumvals        | text[]  |           |          |
 boot_val        | text    |           |          |
 reset_val       | text    |           |          |
 sourcefile      | text    |           |          |
 sourceline      | integer |           |          |
 pending_restart | boolean |           |          |
```
postgres=# `SELECT name, setting FROM pg_settings WHERE name IN ('archive_mode', 'archive_command', 'archive_timeout', 'wal_level', 'max_wal_senders');`
```
      name       |  setting
-----------------+------------
 archive_command | (disabled)
 archive_mode    | off
 archive_timeout | 0
 max_wal_senders | 10
 wal_level       | replica
(5 rows)
```
postgres=# `\q`  

$ `vim postgresql.conf`
```
...
#------------------------------------------------------------------------------
# WRITE-AHEAD LOG
#------------------------------------------------------------------------------

# - Settings -

wal_level = replica             # minimal, replica, or logical
                                # (change requires restart)
...
# - Archiving -

archive_mode = on               # enables archiving; off, on, or always
                                # (change requires restart)
archive_command = 'cp %p /var/lib/pgsql/12/backups/pg_wal/%f'
                                # command to use to archive a logfile segment
                                # placeholders: %p = path of file to archive
                                #               %f = file name only
                                # e.g. 'test ! -f /mnt/server/archivedir/%f && ...

#------------------------------------------------------------------------------
# REPLICATION
#------------------------------------------------------------------------------

# - Sending Servers -

# Set these on the master and on any standby that will send replication data.

max_wal_senders = 2             # max number of walsender processes
                                # (change requires restart)
...

#------------------------------------------------------------------------------
# CUSTOMIZED OPTIONS
#------------------------------------------------------------------------------
```
$ `vim pg_hba.conf`
```
...
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
host    replication     postgres        0.0.0.0/0               md5
host    all             all             0.0.0.0/0               md5
```
$ `pg_ctl stop`  
```
waiting for server to shut down.... done
server stopped
```
$ `pg_ctl start`
```
waiting for server to start....2021-01-26 13:55:42.202 KST [26660] LOG:  starting PostgreSQL 12.5 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-39), 64-bit
2021-01-26 13:55:42.202 KST [26660] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2021-01-26 13:55:42.202 KST [26660] LOG:  listening on IPv6 address "::", port 5432
2021-01-26 13:55:42.203 KST [26660] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2021-01-26 13:55:42.206 KST [26660] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2021-01-26 13:55:42.218 KST [26660] LOG:  redirecting log output to logging collector process
2021-01-26 13:55:42.218 KST [26660] HINT:  Future log output will appear in directory "log".
 done
server started
```
$ `psql`
```
psql (12.5)
Type "help" for help.
```
postgres=# `SELECT name, setting FROM pg_settings WHERE name IN ('archive_mode', 'archive_command', 'archive_timeout', 'wal_level', 'max_wal_senders');`
```
      name       |                setting
-----------------+----------------------------------------
 archive_command | cp %p /var/lib/pgsql/12/backups/pg_wal/%f
 archive_mode    | on
 archive_timeout | 0
 max_wal_senders | 2
 wal_level       | replica
(5 rows)
```
postgres=# `\q`  

$ `ls -al /var/lib/pgsql/12/backups/pg_wal`
```
total 16384
drwxr-xr-x 2 postgres postgres       38 Jan 26 15:23 .
drwx------ 4 postgres postgres       29 Jan 26 15:19 ..
-rw------- 1 postgres postgres 16777216 Jan 26 15:23 000000010000000000000001
```

#### 4.3.2. online 백업 수행  

✔︎ online 백업을 수행합니다. 여기서는 이해를 돕기 위해 테스트 테이블을 생성하고 임의의 장애도 발생 시킵니다.  

$ `ls -al /var/lib/pgsql/12/backups/pg_wal`
```
total 16384
drwxr-xr-x 2 postgres postgres       38 Jan 26 15:23 .
drwx------ 4 postgres postgres       29 Jan 26 15:19 ..
-rw------- 1 postgres postgres 16777216 Jan 26 15:23 000000010000000000000001
```
$ `psql`
```
psql (12.5)
Type "help" for help.
```
postgres=# `create table pitr_test (sn serial, create_date timestamp);`
```
CREATE TABLE
```
postgres=# `select * from pitr_test;`
```
 sn | create_date
----+-------------
(0 rows)
```
postgres=# `\q`  
-bash-4.2$ `crontab -l`  
-bash-4.2$ `crontab -e`
```
* * * * * psql -c "insert into pitr_test(create_date) select now();"
```
```
crontab: installing new crontab
```
-bash-4.2$ `crontab -l`
```
* * * * * psql -c "insert into pitr_test(create_date) select now();"
```
postgres=# `select * from pitr_test;`
```
 sn | create_date
----+-------------
(0 rows)
```

postgres=# `select * from pitr_test;`
```
 sn |        create_date
----+----------------------------
  1 | 2021-01-27 18:54:01.134953
(1 row)
```
postgres=# `select now();`
```
              now
-------------------------------
 2021-01-29 10:51:17.460406+09
(1 row)
```
postgres=# `\q`  
-bash-4.2$ `pg_basebackup -D /var/lib/pgsql/12/backups/basebackup$(date +%Y%m%d)`  
-bash-4.2$ `ls /var/lib/pgsql/12/backups`
```
basebackup20210129  data  offline_data_0129.tar  pg_wal
```
-bash-4.2$ `ls -al /var/lib/pgsql/12/backups/pg_wal`
```
total 131076
drwxr-xr-x 2 postgres postgres      310 Jan 29 10:55 .
drwx------ 5 postgres postgres       87 Jan 29 10:55 ..
-rw------- 1 postgres postgres 16777216 Jan 26 15:23 000000010000000000000001
...
-rw------- 1 postgres postgres 16777216 Jan 29 10:55 000000010000000000000007
-rw------- 1 postgres postgres 16777216 Jan 29 10:55 000000010000000000000008
-rw------- 1 postgres postgres      337 Jan 29 10:55 000000010000000000000008.00000028.backup
```
-bash-4.2$ `ls -al $PGDATA/pg_wal`
```
total 32776
drwx------  3 postgres postgres      140 Jan 29 10:55 .
drwx------ 20 postgres postgres     4096 Jan 29 10:44 ..
-rw-------  1 postgres postgres 16777216 Jan 29 10:55 000000010000000000000008
-rw-------  1 postgres postgres      337 Jan 29 10:55 000000010000000000000008.00000028.backup
-rw-------  1 postgres postgres 16777216 Jan 29 10:57 000000010000000000000009
drwx------  2 postgres postgres       96 Jan 29 10:55 archive_status
```
-bash-4.2$ `psql`
```
psql (12.5)
Type "help" for help.
```
postgres=# `select now();`
```
              now
-------------------------------
 2021-01-29 18:15:19.571685+09
(1 row)
```
postgres=# `select * from pitr_test order by 1 desc limit 5;`
```
  sn  |        create_date
------+----------------------------
 1493 | 2021-01-29 18:15:01.49518
 1492 | 2021-01-29 18:14:01.462195
 1491 | 2021-01-29 18:13:01.429367
 1490 | 2021-01-29 18:12:01.398585
 1489 | 2021-01-29 18:11:01.357323
(5 rows)
```
postgres=# `\q`  
-bash-4.2$ `ps -ef | grep postgres`  
```
root      7901  4882  0 10:28 ?        00:00:00 sshd: postgres [priv]
postgres  7913  7901  0 10:29 ?        00:00:01 sshd: postgres@pts/0
postgres  7914  7913  0 10:29 pts/0    00:00:00 -bash
postgres 19898     1  0 18:05 ?        00:00:00 /usr/pgsql-12/bin/postgres
postgres 19899 19898  0 18:05 ?        00:00:00 postgres: logger
postgres 19901 19898  0 18:05 ?        00:00:00 postgres: checkpointer
postgres 19902 19898  0 18:05 ?        00:00:00 postgres: background writer
postgres 19903 19898  0 18:05 ?        00:00:00 postgres: walwriter
postgres 19904 19898  0 18:05 ?        00:00:00 postgres: autovacuum launcher
postgres 19905 19898  0 18:05 ?        00:00:00 postgres: archiver
postgres 19906 19898  0 18:05 ?        00:00:00 postgres: stats collector
postgres 19907 19898  0 18:05 ?        00:00:00 postgres: logical replication launcher
postgres 20611  7914  0 18:15 pts/0    00:00:00 ps -ef
postgres 20612  7914  0 18:15 pts/0    00:00:00 grep --color=auto postgres
```

-bash-4.2$ `kill -9 19898`  
-bash-4.2$ `ps -ef | grep postgres`
```
root      7901  4882  0 10:28 ?        00:00:00 sshd: postgres [priv]
postgres  7913  7901  0 10:29 ?        00:00:01 sshd: postgres@pts/0
postgres  7914  7913  0 10:29 pts/0    00:00:00 -bash
postgres 20638  7914  0 18:15 pts/0    00:00:00 ps -ef
postgres 20639  7914  0 18:15 pts/0    00:00:00 grep --color=auto postgres
```  

#### 4.3.3. 최근 시점으로 복구  

✔︎ 복원작업 과정에서 원본 파일 손상을 방지하기 위해, wal 파일을 포함한 현재의 data 폴더를 rename하고 online 백업 파일을 복원하여 복구를 수행합니다.  

✔︎ v11까지는 복구 환경 파일로  recovery.conf 파일을 사용하며, 복구 완료 후에는 recovery.done으로 파일명이 자동 변경됩니다. (https://blog.goodusdata.com/12)  

✔︎ v12부터는 recovery.conf 파일 사용 시 복구 작업 오류가 발생하며, recovery.signal 파일과 postgresql.conf 파일을 사용하여 복구합니다. 복구 완료 후 recovery.signal 파일은 자동 삭제됩니다. (https://www.cybertec-postgresql.com/en/recovery-conf-is-gone-in-postgresql-v12/, https://www.postgresql.org/docs/12/continuous-archiving.html)  

-bash-4.2$ `cd /var/lib/pgsql/12/data`  
-bash-4.2$ `ls`
```
backups  data  data.tar  initdb.log
```
-bash-4.2$ `mv data data.bak`  
-bash-4.2$ `ls`  
```
backups  data.bak  data.tar  initdb.log
```
-bash-4.2$ `mkdir data`
-bash-4.2$ `ls -al`
```
total 4704
drwx------  5 postgres postgres      83 Jan 29 11:02 .
drwx------  3 postgres postgres      95 Jan 27 18:32 ..
drwx------  5 postgres postgres      87 Jan 29 10:55 backups
drwxr-xr-x  2 postgres postgres       6 Jan 29 18:15 data
drwx------ 20 postgres postgres    4096 Jan 29 18:15 data.bak
-rw-r--r--  1 postgres postgres 4807145 Jan 25 17:57 data.tar
-rw-------  1 postgres postgres     918 Jan 12 14:49 initdb.log
```
-bash-4.2$ `chmod 700 data`  
-bash-4.2$ `ls -al`  
```
total 4704
drwx------  5 postgres postgres      83 Jan 29 11:02 .
drwx------  3 postgres postgres      95 Jan 27 18:32 ..
drwx------  5 postgres postgres      87 Jan 29 10:55 backups
drwx------  2 postgres postgres       6 Jan 29 11:15 data
drwx------ 20 postgres postgres    4096 Jan 29 18:15 data.bak
-rw-r--r--  1 postgres postgres 4807145 Jan 25 17:57 data.tar
-rw-------  1 postgres postgres     918 Jan 12 14:49 initdb.log
```
-bash-4.2$ `cd /var/lib/pgsql/12/backups/basebackup20210129`  
-bash-4.2$ `cp -ar . /var/lib/pgsql/12/data`  
-bash-4.2$ `ls -al /var/lib/pgsql/12/backups/pg_wal`  
```
total 278536
drwxr-xr-x 2 postgres postgres     4096 Jan 29 18:15 .
drwx------ 5 postgres postgres       58 Jan 29 16:37 ..
-rw------- 1 postgres postgres 16777216 Jan 26 15:23 000000010000000000000001
-rw------- 1 postgres postgres 16777216 Jan 27 07:30 000000010000000000000002
-rw------- 1 postgres postgres 16777216 Jan 27 22:00 000000010000000000000003
-rw------- 1 postgres postgres 16777216 Jan 28 17:26 000000010000000000000004
-rw------- 1 postgres postgres 16777216 Jan 29 07:30 000000010000000000000005
-rw------- 1 postgres postgres 16777216 Jan 29 10:39 000000010000000000000006
-rw------- 1 postgres postgres 16777216 Jan 29 16:31 000000010000000000000007
-rw------- 1 postgres postgres 16777216 Jan 29 16:37 000000010000000000000008
-rw------- 1 postgres postgres 16777216 Jan 29 16:37 000000010000000000000009
-rw------- 1 postgres postgres      337 Jan 29 16:37 000000010000000000000009.00000028.backup
-rw------- 1 postgres postgres 16777216 Jan 29 17:12 00000001000000000000000A
-rw------- 1 postgres postgres 16777216 Jan 29 17:24 00000001000000000000000B
-rw------- 1 postgres postgres 16777216 Jan 29 17:25 00000001000000000000000C
-rw------- 1 postgres postgres 16777216 Jan 29 17:49 00000001000000000000000D
-rw------- 1 postgres postgres 16777216 Jan 29 17:49 00000001000000000000000E
-rw------- 1 postgres postgres 16777216 Jan 29 17:55 00000001000000000000000F
-rw------- 1 postgres postgres 16777216 Jan 29 18:03 000000010000000000000010
-rw------- 1 postgres postgres 16777216 Jan 29 18:15 000000010000000000000011
```
-bash-4.2$ `ls -al /var/lib/pgsql/12/data/pg_wal`  
```
total 32776
drwx------  3 postgres postgres      140 Jan 29 18:27 .
drwx------ 20 postgres postgres     4096 Jan 29 18:12 ..
-rw-------  1 postgres postgres 16777216 Jan 29 16:37 000000010000000000000009
-rw-------  1 postgres postgres      337 Jan 29 16:37 000000010000000000000009.00000028.backup
-rw-------  1 postgres postgres 16777216 Jan 29 16:38 00000001000000000000000A
drwx------  2 postgres postgres        6 Jan 29 16:37 archive_status
```
-bash-4.2$ `ls -al /var/lib/pgsql/12/data.bak/pg_wal`  
```
total 49160
drwx------  3 postgres postgres      172 Jan 29 18:09 .
drwx------ 20 postgres postgres     4096 Jan 29 18:15 ..
-rw-------  1 postgres postgres      337 Jan 29 16:37 000000010000000000000009.00000028.backup
-rw-------  1 postgres postgres 16777216 Jan 29 18:15 000000010000000000000011
drwx------  2 postgres postgres       43 Jan 29 18:15 archive_status
```
-bash-4.2$ `cp -p /var/lib/pgsql/12/data.bak/pg_wal/0* /var/lib/pgsql/12/data/pg_wal`  
-bash-4.2$ `ls -al /var/lib/pgsql/12/data/pg_wal`  
```
total 49160
drwx------  3 postgres postgres      140 Jan 29 18:27 .
drwx------ 20 postgres postgres     4096 Jan 29 18:12 ..
-rw-------  1 postgres postgres 16777216 Jan 29 16:37 000000010000000000000009
-rw-------  1 postgres postgres      337 Jan 29 16:37 000000010000000000000009.00000028.backup
-rw-------  1 postgres postgres 16777216 Jan 29 16:38 00000001000000000000000A
-rw-------  1 postgres postgres 16777216 Jan 29 18:15 000000010000000000000011
drwx------  2 postgres postgres       43 Jan 29 18:15 archive_status
```
-bash-4.2$ `cd $PGDATA`  
-bash-4.2$ `vi recovery.conf`  
-bash-4.2$ `cat recovery.conf`  
```
restore_command = 'cp /var/lib/pgsql/12/backups/pg_wal/%f %p'
```
-bash-4.2$ `pg_ctl start`  
```
waiting for server to start....2021-01-29 18:29:10.512 KST [21460] LOG:  starting PostgreSQL 12.5 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-39), 64-bit
2021-01-29 18:29:10.512 KST [21460] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2021-01-29 18:29:10.512 KST [21460] LOG:  listening on IPv6 address "::", port 5432
2021-01-29 18:29:10.513 KST [21460] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2021-01-29 18:29:10.516 KST [21460] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2021-01-29 18:29:10.531 KST [21460] LOG:  redirecting log output to logging collector process
2021-01-29 18:29:10.531 KST [21460] HINT:  Future log output will appear in directory "log".
 stopped waiting
pg_ctl: could not start server
Examine the log output.
```
-bash-4.2$ `tail -f postgresql-Fri.log`  
```
2021-01-29 16:34:16.464 KST [14912] LOG:  database system was not properly shut down; automatic recovery in progress
2021-01-29 16:34:16.465 KST [14912] LOG:  redo starts at 0/80000A0
2021-01-29 16:34:16.466 KST [14912] LOG:  invalid record length at 0/8001A80: wanted 24, got 0
2021-01-29 16:34:16.466 KST [14912] LOG:  redo done at 0/8001A48
2021-01-29 16:34:16.490 KST [14910] LOG:  database system is ready to accept connections
2021-01-29 18:29:10.534 KST [21462] LOG:  database system was interrupted; last known up at 2021-01-29 16:37:05 KST
2021-01-29 18:29:10.547 KST [21462] FATAL:  using recovery command file "recovery.conf" is not supported
2021-01-29 18:29:10.548 KST [21460] LOG:  startup process (PID 21462) exited with exit code 1
2021-01-29 18:29:10.548 KST [21460] LOG:  aborting startup due to startup process failure
2021-01-29 18:29:10.550 KST [21460] LOG:  database system is shut down
```
-bash-4.2$ `cp postgresql.conf postgresql.conf.20210129`  
-bash-4.2$ `vi postgresql.conf`  
```
...
restore_command = 'cp /var/lib/pgsql/12/backups/pg_wal/%f %p'
                        # command to use to restore an archived logfile segment
                        # placeholders: %p = path of file to restore
                        #               %f = file name only
                        # e.g. 'cp /mnt/server/archivedir/%f %p'
                        # (change requires restart)
...
```
-bash-4.2$ `rm recovery.conf`  
-bash-4.2$ `touch recovery.signal`  
-bash-4.2$ `ls -al`  
```
total 104
...
-rw-------  1 postgres postgres 26737 Jan 29 17:48 postgresql.conf
-rw-------  1 postgres postgres 26626 Jan 29 16:37 postgresql.conf.20210113
-rw-------  1 postgres postgres    27 Jan 29 18:29 postmaster.opts
-rw-r--r--  1 postgres postgres     0 Jan 29 18:35 recovery.signal
```
-bash-4.2$ `pg_ctl start`  
```
waiting for server to start....2021-01-29 18:35:35.767 KST [21937] LOG:  starting PostgreSQL 12.5 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-39), 64-bit
2021-01-29 18:35:35.767 KST [21937] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2021-01-29 18:35:35.767 KST [21937] LOG:  listening on IPv6 address "::", port 5432
2021-01-29 18:35:35.768 KST [21937] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2021-01-29 18:35:35.771 KST [21937] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2021-01-29 18:35:35.786 KST [21937] LOG:  redirecting log output to logging collector process
2021-01-29 18:35:35.786 KST [21937] HINT:  Future log output will appear in directory "log".
 done
server started
```
-bash-4.2$ `tail -f postgresql-Fri.log`
```
2021-01-29 18:35:35.806 KST [21939] LOG:  starting archive recovery
2021-01-29 18:35:35.899 KST [21939] LOG:  restored log file "000000010000000000000009" from archive
2021-01-29 18:35:35.954 KST [21939] LOG:  redo starts at 0/9000028
2021-01-29 18:35:35.956 KST [21939] LOG:  consistent recovery state reached at 0/9000100
2021-01-29 18:35:35.956 KST [21937] LOG:  database system is ready to accept read only connections
2021-01-29 18:35:36.057 KST [21939] LOG:  restored log file "00000001000000000000000A" from archive
2021-01-29 18:35:36.217 KST [21939] LOG:  restored log file "00000001000000000000000B" from archive
...
...
...
2021-01-29 18:35:37.340 KST [21939] LOG:  redo done at 0/11001DD8
2021-01-29 18:35:37.340 KST [21939] LOG:  last completed transaction was at log time 2021-01-29 18:15:01.495664+09
2021-01-29 18:35:37.355 KST [21939] LOG:  restored log file "000000010000000000000011" from archive
```
-bash-4.2$ `ls -al recovery.signal`  
```
ls: cannot access postgresql.conf.202101131: No such file or directory
```
-bash-4.2$ `psql`
```
psql (12.5)
Type "help" for help.
```

postgres=# `select * from pitr_test order by 1 desc limit 5;`
```
  sn  |        create_date
------+----------------------------
 1527 | 2021-01-29 18:37:01.2624
 1526 | 2021-01-29 18:36:01.188296
 1493 | 2021-01-29 18:15:01.49518
 1492 | 2021-01-29 18:14:01.462195
 1491 | 2021-01-29 18:13:01.429367
(5 rows)
```

#### 4.3.4. AWS RDS 백업 및 복원  
  
✔︎ AWS RDS는 snapshot 형식으로 백업과 복원을 지원합니다. 백업은 자동 스케줄 백업 및 수동 백업을 지원합니다. 복구는 snapshot 백업본에서 새로운 instance를 생성하는 방법으로 지원합니다.  
✔︎ 자동 snapshot 백업본에서 DB 인스턴스의 복원 가능한 최근 시간은 일반적으로 최근 5분이며, 원하는 시점의 복원도 가능합니다(최대 35일 이내). 만약 RDS 엔진으로 Aurora를 사용할 경우에는 공유 스토리지 방식을 사용하여 일반적으로 알려진 데이터 손실이 발생하지 않습니다.  

✔︎ snapshot 백업

* AWS RDS의 databases 메뉴에서 Take snapshot Action을 선택하여 수행합니다. 참고로, 자동 백업본은 instance 삭제와 함께 삭제되며, Manual 백업본은 사용자가 직접 삭제해야 합니다.

* 이미지 파일 "aws_rds_snapshot_backup_01.jpg" 링크 예정
* 이미지 파일 "aws_rds_snapshot_backup_02.jpg" 링크 예정
* 이미지 파일 "aws_rds_snapshot_backup_03.jpg" 링크 예정

✔︎ snapshot 복원

* AWS RDS의 databases 메뉴에서 Restore to point in time (또는 UI에 따라 Restore Snapshot) Action을 선택하여 수행합니다. 최신 Snapshot 또는 사용자 지정 시간 옵션을 선택한 후 새롭게 생성할 인스턴스 타입과 구성을 적절하게 지정합니다. 최종적으로 제일 하단의 Restore DB Cluster 버튼을 클릭하여 복원 작업을 완료합니다.

* 이미지 파일 "aws_rds_snapshot_restore_01.jpg" 링크 예정
* 이미지 파일 "aws_rds_snapshot_restore_02.jpg" 링크 예정
* 이미지 파일 "aws_rds_snapshot_restore_03.jpg" 링크 예정
* 이미지 파일 "aws_rds_snapshot_restore_04.jpg" 링크 예정
* 이미지 파일 "aws_rds_snapshot_restore_05.jpg" 링크 예정  

## 5. AWS DB 참고 사항

### 5.1. Serverless 비용 주의

✔︎ 비용 절감을 위해 Serverless RDS를 사용할 경우에 idle connection에 주의해야 합니다.비록 Active Session이 아니어도 1개라도 세션이 존재한다면 instance 자동 Shutdown이 수행되지 않아서 비용을 지불해야 합니다. 결과적으로 WAS connection pool 상시 사용은 Serverless에 적합하지 않습니다.  

### 5.2. DynamoDB 비용 주의  

✔︎ 프로비저닝된 용량 모드로 테이블을 사용할 경우에 테이블의 삭제 및 재생성 테스트에 주의해야 합니다. 프로비저닝된 용량 모드는 적정 성능을 예약하는 경우이기 때문에 한 번 생성되는 경우 기본 요금으로 1시간 단위 용량이 청구됩니다. 즉 테스트를 위해 반복적으로 재생성되는 테스트 횟수에 1시간 비용을 곱한 수 만큼 요금이 청구됩니다.

### 5.3. DynamoDB 접근 제어  

✔︎ DynamoDB는 일반적인 RDB와 같이 DB Object 접근제어에 사용하는 DB 내부 계정을 갖고 있지 않습니다.  
✔︎ DynamoDB는 자격 증명 기반 정책(identity-based policies, IAM policies)으로만 접근제어를 지원하며, 리소스 기반 정책(resource-based policies)은 지원하지 않습니다.  
✔︎ [DynamoDB 특정 테이블 액세스 허용 Sample](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/reference_policies_examples_dynamodb_specific-table.html)  
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAndDescribe",
            "Effect": "Allow",
            "Action": [
                "dynamodb:List*",
                "dynamodb:DescribeReservedCapacity*",
                "dynamodb:DescribeLimits",
                "dynamodb:DescribeTimeToLive"
            ],
            "Resource": "*"
        },
        {
            "Sid": "SpecificTable",
            "Effect": "Allow",
            "Action": [
                "dynamodb:BatchGet*",
                "dynamodb:DescribeStream",
                "dynamodb:DescribeTable",
                "dynamodb:Get*",
                "dynamodb:Query",
                "dynamodb:Scan",
                "dynamodb:BatchWrite*",
                "dynamodb:CreateTable",
                "dynamodb:Delete*",
                "dynamodb:Update*",
                "dynamodb:PutItem"
            ],
            "Resource": "arn:aws:dynamodb:*:*:table/MyTable"
        }
    ]
}
```  
✔︎ [DynamoDB 특정 속성 액세스 허용 Sample](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/reference_policies_examples_dynamodb_attributes.html)
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:GetItem",
                "dynamodb:BatchGetItem",
                "dynamodb:Query",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem",
                "dynamodb:BatchWriteItem"
            ],
            "Resource": ["arn:aws:dynamodb:*:*:table/table-name"],
            "Condition": {
                "ForAllValues:StringEquals": {
                    "dynamodb:Attributes": [
                        "column-name-1",
                        "column-name-2",
                        "column-name-3"
                    ]
                },
                "StringEqualsIfExists": {"dynamodb:Select": "SPECIFIC_ATTRIBUTES"}
            }
        }
    ]
}
```
✔︎ [참고: Amazon DynamoDB 서울 리전 엔드포인트: dynamodb.ap-northeast-2.amazonaws.com](https://docs.aws.amazon.com/ko_kr/general/latest/gr/ddb.html)  

### 5.4. Python AWS Access code  

✔︎ Python Windows Installer 다운로드 및 설치
* https://www.python.org/ 이동합니다.  
* Downloads 메뉴에서 현재 version (3.9.1) 선택합니다.  
* 다운로드 완료 후에 설치합니다.

✔︎ python Editor 중에서 Visual Studio code 다운로드 및 설치
* https://code.visualstudio.com/#alt-downloads 이동합니다.
* Windows System installer 64bit 선택합니다.
* 다운로드 완료 후에 설치합니다.

✔︎ AWS Python SDK(Boto3) 다운로드 및 설치
* Windows PC의 Python 설치 경로에서 python install 명령어를 수행하면 python source code에서 import boto3 수행이 가능해집니다.

c:\Python>`python -m pip install boto3`  
```
Collecting boto3
  Downloading boto3-1.17.1.tar.gz (100 kB)
     |████████████████████████████████| 100 kB 1.2 MB/s
Collecting botocore<1.21.0,>=1.20.1
  Downloading botocore-1.20.1-py2.py3-none-any.whl (7.2 MB)
     |████████████████████████████████| 7.2 MB 6.4 MB/s
Collecting jmespath<1.0.0,>=0.7.1
  Downloading jmespath-0.10.0-py2.py3-none-any.whl (24 kB)
Collecting s3transfer<0.4.0,>=0.3.0
  Downloading s3transfer-0.3.4-py2.py3-none-any.whl (69 kB)
     |████████████████████████████████| 69 kB 4.8 MB/s
Collecting python-dateutil<3.0.0,>=2.1
  Downloading python_dateutil-2.8.1-py2.py3-none-any.whl (227 kB)
     |████████████████████████████████| 227 kB ...
Collecting urllib3<1.27,>=1.25.4
  Downloading urllib3-1.26.3-py2.py3-none-any.whl (137 kB)
     |████████████████████████████████| 137 kB 6.8 MB/s
Requirement already satisfied: six>=1.5 in c:\users\askto\appdata\roaming\python\python39\site-packages (from python-dateutil<3.0.0,>=2.1->botocore<1.21.0,>=1.20.1->boto3) (1.15.0)
Using legacy 'setup.py install' for boto3, since package 'wheel' is not installed.
Installing collected packages: jmespath, python-dateutil, urllib3, botocore, s3transfer, boto3
    Running setup.py install for boto3 ... done
Successfully installed boto3-1.17.1 botocore-1.20.1 jmespath-0.10.0 python-dateutil-2.8.1 s3transfer-0.3.4 urllib3-1.26.3

c:\Python\Python39>python --version
Python 3.9.1

c:\Python\Python39>pip --version
pip 21.0.1 from c:\python\python39\lib\site-packages\pip (python 3.9)
``` 
✔︎ Python Code 실행 시 AWS 서비스 접속을 위한 자격 증명 설정  

* AWS security credentials 설정으로 boto3 api 동작에 필요한 자격 증명 정보를 제공합니다. 
* 설정은 구성 파일 또는 Hard Coding의 2가지 방법 중에서 선택하여 사용합니다.

✔︎ 자격 증명 구성 파일 Sample
* 구성 파일은 "credentials" 파일명으로 지정 경로에 생성합니다.
* 지정 경로는 리눅스에서 "~/.aws/" 윈도우에서 "C:\Users\USER_NAME\\.aws\\" 사용합니다.  

```
[default]
aws_access_key_id = <your access key id>
aws_secret_access_key = <your secret key>
...
```

✔︎ Python Source Code 자격 증명 Sample
```
…
s3client = boto3.client('s3', aws_access_key_id='...',aws_secret_access_key='...')
…
s3resource = boto3.resource('s3', aws_access_key_id='...',aws_secret_access_key='...')
…
dynamodb = boto3.resource('dynamodb', region_name='ap-northeast-2', aws_access_key_id='...',aws_secret_access_key='...')
...
```

✔︎ boto3 API 스타일
* boto3는 Resource API (high-level)와 Client API (low-level)의 두 가지 API 스타일을 제공합니다.
* Resource API는 Client API를 추상화하여 코드를 단순화하는 데 도움이 되지만 Client API에 대한 모든 기능을 제공하지 않습니다. 따라서 두 가지 API 스타일을 같이 사용하는 경우가 생길 수 있습니다.
* AWS 신규 고객의 경우 사용중인 서비스에 리소스 API로 시작하는 것이 좋습니다.  

✔︎ [Python Sample Code](https://github.com/aws-samples/aws-python-sample)
```
# Import the SDK
import boto3
import uuid

#    1. Environment variables (AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY)
#    2. Credentials file (~/.aws/credentials or
#         C:\Users\USER_NAME\.aws\credentials)
#    3. AWS IAM role for Amazon EC2 instance
#       (http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)

s3client = boto3.client('s3')

# Everything uploaded to Amazon S3 must belong to a bucket. These buckets are
# in the global namespace, and must have a unique name.

bucket_name = 'python-sdk-sample-{}'.format(uuid.uuid4())
print('Creating new bucket with name: {}'.format(bucket_name))
s3client.create_bucket(Bucket=bucket_name)

# Now the bucket is created, and you'll find it in your list of buckets.

list_buckets_resp = s3client.list_buckets()
for bucket in list_buckets_resp['Buckets']:
    if bucket['Name'] == bucket_name:
        print('(Just created) --> {} - there since {}'.format(
            bucket['Name'], bucket['CreationDate']))

# Files in Amazon S3 are called "objects" and are stored in buckets. A
# specific object is referred to by its key (i.e., name) and holds data. Here,
# we create (put) a new object with the key "python_sample_key.txt" and
# content "Hello World!".

object_key = 'python_sample_key.txt'

print('Uploading some data to {} with key: {}'.format(
    bucket_name, object_key))
s3client.put_object(Bucket=bucket_name, Key=object_key, Body=b'Hello World!')

# Using the client, you can generate a pre-signed URL that you can give
# others to securely share the object without making it publicly accessible.
# By default, the generated URL will expire and no longer function after one
# hour. You can change the expiration to be from 1 second to 604800 seconds
# (1 week).

url = s3client.generate_presigned_url(
    'get_object', {'Bucket': bucket_name, 'Key': object_key})
print('\nTry this URL in your browser to download the object:')
print(url)

try:
    input = raw_input
except NameError:
    pass
input("\nPress enter to continue...")

# As we've seen in the create_bucket, list_buckets, and put_object methods,
# Client API requires you to explicitly specify all the input parameters for
# each operation. Most methods in the client class map to a single underlying
# API call to the AWS service - Amazon S3 in our case.
#
# Now that you got the hang of the Client API, let's take a look at Resouce
# API, which provides resource objects that further abstract out the over-the-
# network API calls.
# Here, we'll instantiate and use 'bucket' or 'object' objects.

print('\nNow using Resource API')
# First, create the service resource object
s3resource = boto3.resource('s3')
# Now, the bucket object
bucket = s3resource.Bucket(bucket_name)
# Then, the object object
obj = bucket.Object(object_key)
print('Bucket name: {}'.format(bucket.name))
print('Object key: {}'.format(obj.key))
print('Object content length: {}'.format(obj.content_length))
print('Object body: {}'.format(obj.get()['Body'].read()))
print('Object last modified: {}'.format(obj.last_modified))

# Buckets cannot be deleted unless they're empty. Let's keep using the
# Resource API to delete everything. Here, we'll utilize the collection
# 'objects' and its batch action 'delete'. Batch actions return a list
# of responses, because boto3 may have to take multiple actions iteratively to
# complete the action.

print('\nDeleting all objects in bucket {}.'.format(bucket_name))
delete_responses = bucket.objects.delete()
for delete_response in delete_responses:
    for deleted in delete_response['Deleted']:
        print('\t Deleted: {}'.format(deleted['Key']))

# Now that the bucket is empty, let's delete the bucket.

print('\nDeleting the bucket.')
bucket.delete()

# For more details on what you can do with boto3 and Amazon S3, see the API
# reference page:
# https://boto3.readthedocs.org/en/latest/reference/services/s3.html
```  
✔︎ [참고: AWS 리전별 제공 서비스 목록과 엔드포인트 쉽게 가져오는 방법](https://aws.amazon.com/ko/blogs/korea/new-query-for-aws-regions-endpoints-and-more-using-aws-systems-manager-parameter-store/)  

```
Pelican

C:\Users\askto>pip install virtualenv
Collecting virtualenv
  Downloading virtualenv-20.4.2-py2.py3-none-any.whl (7.2 MB)
     |████████████████████████████████| 7.2 MB 2.2 MB/s
Collecting filelock<4,>=3.0.0
  Downloading filelock-3.0.12-py3-none-any.whl (7.6 kB)
Collecting appdirs<2,>=1.4.3
  Downloading appdirs-1.4.4-py2.py3-none-any.whl (9.6 kB)
Requirement already satisfied: six<2,>=1.9.0 in c:\users\askto\appdata\roaming\python\python39\site-packages (from virtualenv) (1.15.0)
Collecting distlib<1,>=0.3.1
  Downloading distlib-0.3.1-py2.py3-none-any.whl (335 kB)
     |████████████████████████████████| 335 kB 6.4 MB/s
Installing collected packages: filelock, distlib, appdirs, virtualenv
Successfully installed appdirs-1.4.4 distlib-0.3.1 filelock-3.0.12 virtualenv-20.4.2

C:\Users\askto>pip install pelican markdown
Collecting pelican
  Downloading pelican-4.5.4-py3-none-any.whl (1.4 MB)
     |████████████████████████████████| 1.4 MB 2.2 MB/s
Collecting markdown
  Downloading Markdown-3.3.3-py3-none-any.whl (96 kB)
     |████████████████████████████████| 96 kB 2.6 MB/s
Collecting feedgenerator<2.0,>=1.9
  Downloading feedgenerator-1.9.1-py3-none-any.whl (22 kB)
Requirement already satisfied: python-dateutil<3.0,>=2.8 in c:\python\python39\lib\site-packages (from pelican) (2.8.1)
Collecting jinja2<2.12,>=2.11
  Downloading Jinja2-2.11.3-py2.py3-none-any.whl (125 kB)
     |████████████████████████████████| 125 kB 6.4 MB/s
Collecting blinker<2.0,>=1.4
  Downloading blinker-1.4.tar.gz (111 kB)
     |████████████████████████████████| 111 kB ...
Collecting docutils<0.17,>=0.16
  Downloading docutils-0.16-py2.py3-none-any.whl (548 kB)
     |████████████████████████████████| 548 kB 6.4 MB/s
Collecting unidecode<2.0,>=1.1
  Downloading Unidecode-1.2.0-py2.py3-none-any.whl (241 kB)
     |████████████████████████████████| 241 kB 6.4 MB/s
Collecting pytz<2021.0,>=2020.1
  Downloading pytz-2020.5-py2.py3-none-any.whl (510 kB)
     |████████████████████████████████| 510 kB 6.8 MB/s
Collecting pygments<2.7.0,>=2.6.1
  Downloading Pygments-2.6.1-py3-none-any.whl (914 kB)
     |████████████████████████████████| 914 kB 6.8 MB/s
Requirement already satisfied: six in c:\users\askto\appdata\roaming\python\python39\site-packages (from feedgenerator<2.0,>=1.9->pelican) (1.15.0)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1-cp39-cp39-win_amd64.whl (16 kB)
Using legacy 'setup.py install' for blinker, since package 'wheel' is not installed.
Installing collected packages: pytz, MarkupSafe, unidecode, pygments, jinja2, feedgenerator, docutils, blinker, pelican, markdown
    Running setup.py install for blinker ... done
Successfully installed MarkupSafe-1.1.1 blinker-1.4 docutils-0.16 feedgenerator-1.9.1 jinja2-2.11.3 markdown-3.3.3 pelican-4.5.4 pygments-2.6.1 pytz-2020.5 unidecode-1.2.0

C:\Python\asktom21\pelican>pelican-quickstart
Welcome to pelican-quickstart v4.5.4.

This script will help you create a new Pelican-based website.

Please answer the following questions so this script can generate the files
needed by Pelican.


> Where do you want to create your new web site? [.]
> What will be the title of this web site? asktom21-pelican
> Who will be the author of this web site? asktom21
> What will be the default language of this web site? [Korean]
> Do you want to specify a URL prefix? e.g., https://example.com   (Y/n) n
> Do you want to enable article pagination? (Y/n)
> How many articles per page do you want? [10]
> What is your time zone? [Europe/Paris] Asia/Seoul
> Do you want to generate a tasks.py/Makefile to automate generation and publishing? (Y/n)
> Do you want to upload your website using FTP? (y/N)
> Do you want to upload your website using SSH? (y/N)
> Do you want to upload your website using Dropbox? (y/N)
> Do you want to upload your website using S3? (y/N)
> Do you want to upload your website using Rackspace Cloud Files? (y/N)
> Do you want to upload your website using GitHub Pages? (y/N)
Done. Your new project is available at C:\Python\asktom21\pelican

C:\Python\asktom21\pelican>explorer .

C:\Python\asktom21\pelican>
```
