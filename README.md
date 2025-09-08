# MYSQL_Backup
Linux Ubuntu 환경에서 crontab을 이용한 MYSQL database 백업 자동화
## 🔹 1. MySQL 설치 (Ubuntu)

1. 패키지 목록 업데이트
    
    ```bash
    sudo apt update
    
    ```
    
2. MySQL 서버 설치
    
    ```bash
    sudo apt install mysql-server -y
    
    ```
    
3. MySQL 접속
    
    ```bash
    sudo mysql
    
    ```
    
    - Ubuntu 기본 root 계정은 `auth_socket` 인증을 사용합니다.
    - 비밀번호 인증으로 바꾸려면:
        
        ```sql
        ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '비밀번호';
        FLUSH PRIVILEGES;
        
        ```
        
    - 이후 접속:
        
        ```bash
        mysql -u root -p비밀번호
        
        ```
        
4. 서비스 관리
    - 상태 확인:
        
        ```bash
        systemctl status mysql
        
        ```
        
    - 시작:
        
        ```bash
        sudo systemctl start mysql
        
        ```
        
    - 자동 실행 활성화:
        
        ```bash
        sudo systemctl enable mysql
        
        ```
        

---

## 🔹 2. DB 덤프 수동 생성

1. 백업 디렉터리 생성
    
    ```bash
    sudo mkdir -p /database
    sudo chown ubuntu:ubuntu /database
    
    ```
    
2. 수동 덤프 생성
    
    ```bash
    mysqldump -u root -p비밀번호 fisa > /database/db.sql
    
    ```
    
    - `fisa` : 백업할 데이터베이스 이름
    - `비밀번호` : MySQL root 계정 비밀번호

---

## 🔹 3. 백업 자동화 스크립트

1. 스크립트 작성 (`/home/ubuntu/database/db_dump.sh`)
    
    ```bash
    #!/bin/bash
    
    DATE=$(date +%Y%m%d)
    BACKUP_DIR="/home/ubuntu/database"
    DB_NAME="fisa"
    DB_USER="root"
    DB_PASS="비밀번호"
    
    # SQL 덤프 파일 이름
    SQL_FILE=$BACKUP_DIR/db_${DATE}.sql
    # 압축 파일 이름
    TAR_FILE=$BACKUP_DIR/db_${DATE}.tar.gz
    
    # DB 덤프 실행
    mysqldump -u$DB_USER -p$DB_PASS $DB_NAME > $SQL_FILE
    
    # tar.gz 로 압축
    tar -czf $TAR_FILE -C $BACKUP_DIR $(basename $SQL_FILE)
    
    # 원본 SQL 파일 삭제
    rm -f $SQL_FILE
    
    # 성공 메시지 출력
    echo "✅ 백업이 성공적으로 완료되었습니다: $TAR_FILE"
    
    ```
    
2. 실행 권한 부여
    
    ```bash
    chmod +x /home/ubuntu/database/db_dump.sh
    
    ```
    
3. 실행 테스트
    
    ```bash
    ./db_dump.sh
    
    ```
    
    실행 후 `/home/ubuntu/database` 안에 `db_YYYYMMDD.tar.gz` 형식의 파일이 생성됩니다.
    
    <img width="889" height="203" alt="image" src="https://github.com/user-attachments/assets/47a9dfbb-d76b-4365-b638-f053e0b886f3" />

    

---

## 🔹 4. 크론탭으로 자동 실행

1. 크론탭 편집
    
    ```bash
    crontab -e
    
    ```
    
2. 아래 내용 추가 (매일 새벽 2시 실행)
    
    ```
    0 2 * * * /home/ubuntu/database/db_dump.sh >> /home/ubuntu/db_backup.log 2>&1
    
    ```
    
    - `0 2 * * *` → 매일 02:00 실행
    - `>> /home/ubuntu/db_backup.log 2>&1` → 실행 결과와 에러를 `db_backup.log`에 기록

---

## 🔹 5. 복원 방법

1. tar 파일 풀기
    
    ```bash
    tar -xvzf db_YYYYMMDD.tar.gz
    
    ```
    
    <img width="725" height="55" alt="image 1" src="https://github.com/user-attachments/assets/e0056830-beb8-428e-a822-7a782b22bd7a" />

    

1. DB에 복원
    
    ```bash
    mysql -u root -p fisa < db_YYYYMMDD.sql
    ```
    

---
