1. RDS 데이터베이스 추가
    
    <aside>
    
    `데이터베이스  이름:` cicd_test_db 
    `마스터 사용자 이름:` 아이디
    `마스터 사용자 암호:` 비밀번호
    
    </aside>
    
2. EC2 인스턴스 추가
    - cicd-test 인스턴스 생성 및 실행
    - 보안 그룹 설정 (포트: 22, 8080, 3306 등 필요에 따라 허용)
3. GitHub Actions Secrets 등록
    - `EC2_HOST`, `EC2_USER`, `EC2_KEY`, `APP_PATH` 등록
4. EC2 내 CICD 실행 환경 구축
    - Docker, docker-compose 설치
    - docker-compose 설치
5. GitHub Actions로 jar 및 Docker 파일 전송
    - `.github/workflows/deploy.yml` 작성
    - scp 통해 EC2에 build/libs/*.jar, Dockerfile, docker-compose.yml 전송
    - ssh로 접속 후 `docker compose up --build -d` 명령 실행
6. SpringBoot 프로젝트 생성 및 gradle 설정
    - build/libs에 jar 생성 확인
    - `application.yml`에서 DB 연결 확인
7. Workflow 트리거 테스트
    - main 브랜치로 push 시 workflow 작동
    - 자동 배포 정상 완료 확인
8. 최종 구현 완료
    - GitHub Actions ↔ AWS EC2 연동 완료
    - 푸시 트리거 기반 자동 빌드/배포 확인

## ✅ 작동 원리

### 요약

1. `main` 브랜치에 코드가 push되면 → GitHub Actions 워크플로우가 자동 실행됨
2. GitHub Actions가 코드 빌드 & JAR 생성 → EC2 서버에 파일 전송
3. EC2 서버에서 Docker Compose로 앱 재배포

---

### 0️⃣ 동작 목표

> 수동 배포 없이 코드 수정만으로 자동으로 테스트와 배포까지 완료되도록 하기 위함
> 

### 1️⃣ **트리거 조건**

- `main` 브랜치에 push 되면 자동 실행됨
- `workflow_dispatch:`로 수동 실행 가능

### 2️⃣ **Build 단계 (GitHub Actions 내부에서 실행)**

| 순서 | 단계 | 설명 |
| --- | --- | --- |
| 1 | Checkout | GitHub에서 소스코드 내려받기 |
| 2 | Gradle 캐시 | Gradle 종속성 다운로드 시간을 줄이기 위해 캐시 적용 |
| 3 | JDK 설정 | Java 17 설치 |
| 4 | Gradle 빌드 | `./gradlew clean build` → `build/libs/*.jar` 생성 |
| 5 | 결과 업로드 | JAR, Dockerfile, docker-compose.yml 파일을 아티팩트로 업로드 |

### 3️⃣ **Deploy 단계 (EC2에 파일 전송 및 배포)**

> needs: build → 빌드 성공 시에만 실행됨
> 

| 순서 | 단계 | 설명 |
| --- | --- | --- |
| 1 | 아티팩트 다운로드 | 앞에서 업로드한 `build-artifacts`를 다시 받아옴 |
| 2 | SCP 전송 | EC2 서버로 `.jar`, Dockerfile, docker-compose.yml 전송 (SSH 키 사용) |
| 3 | SSH 접속 & 배포 | EC2 내에서 아래 순서로 자동 실행됨: |

## ✅ [테스트]

### 1️⃣ 새 리포지토리 생성

### 2️⃣ GitHub Actions Secrets 추가

> Repository > Settings > Secrets And variables > Actions
> 

<aside>

`Name:` APP_PATH  `Secret:` /home/ec2-user/app

`Name:` EC2_HOST  `Secret:` ec2 의 퍼블릭 주소

`Name:` EC2_USER  `Secret:` ec2-user

`Name:` EC2_KEY  `Secret:` cicd-test.pem 파일 오픈 > 내용 모두 복사 후 붙여넣기

[cicd-test.pem](attachment:7304057f-220f-4457-a702-39c148f980b7:cicd-test.pem)

</aside>

### 3️⃣ 리포지토리 환경 구성

```powershell
# 1. 프로젝트 디렉토리로 이동

# 2. 해당 경로에서 git bash 실행

# 3. Git 초기화
git init

# 4. main 브랜치 생성 및 이동 (workflow 실행이 main 브랜치 기준)
git checkout -b main

# 5. GitHub에 새 레포지토리 만든 후 원격 저장소 등록
git remote add origin https://github.com/사용자이름/레포이름.git
```

### 4️⃣ 테스트

```powershell
# 6. 파일 스테이징 및 커밋
git add .
git commit -m "Initial commit on main"

# 7. main 브랜치로 푸시
git push -u origin main

# GitHub Actions 탭 들어가서 workflow 동작 확인 (build -> deploy)
```

### 5️⃣ 결과 이미지 및 시연 영상 링크

![image.png](attachment:38e477dd-7d3d-4ee9-b0c0-e28565e51648:image.png)

https://www.canva.com/design/DAGkkXl20Z8/-hMIfDPEanakH_-Q1yEZSQ/view?utm_content=DA[…]hare&utm_medium=link2&utm_source=uniquelinks&utlId=h56255d4ad4
