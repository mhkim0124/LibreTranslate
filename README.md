# LibreTranslate - 번역 서비스

오픈 소스 기계 번역 API 서비스입니다. 한국어, 영어, 스페인어, 일본어, 중국어 번역을 지원합니다.

## 필요한 프로그램

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- 최소 4GB 이상의 디스크 공간
- 2GB 이상의 RAM 권장

## 실행 방법 (Docker 사용 - 권장)

### 1. 저장소 복제

```bash
git clone https://github.com/mhkim0124/LibreTranslate.git
cd LibreTranslate
```

### 2. Docker Compose로 서비스 실행

```bash
docker-compose up -d
```

위 명령어를 실행하면:
- LibreTranslate Docker 이미지를 다운로드합니다
- 언어 모델(en, ko, es, ja, zh)을 다운로드합니다
- 3006 포트로 서비스가 실행됩니다
- 언어 모델을 저장할 볼륨이 생성됩니다

**참고**: Docker를 사용하면 Python 환경 설정이 필요 없습니다. 모든 의존성이 Docker 이미지에 포함되어 있습니다.

### 3. 서비스 접속

- **웹 인터페이스**: 브라우저에서 `http://localhost:3006` 접속
- **API 엔드포인트**: `http://localhost:3006/translate`
- **헬스 체크**: `http://localhost:3006/health`

### 4. 서비스 관리

```bash
# 로그 확인
docker-compose logs -f

# 실행 중인 컨테이너 확인
docker ps

# 서비스 중지
docker-compose down

# 서비스 중지 및 볼륨 삭제 (다운로드한 모델도 삭제됨)
docker-compose down -v
```

## 설정

`docker-compose.yml` 파일에서 다음 설정을 변경할 수 있습니다:

### 포트 설정
- **호스트 포트**: 3006
- **컨테이너 포트**: 5000

### 언어 모델
```yaml
LT_LOAD_ONLY=en,ko,es,ja,zh
```
- en: 영어
- ko: 한국어
- es: 스페인어
- ja: 일본어
- zh: 중국어

### 품질 설정
```yaml
LT_QUALITY_LEVEL=high  # basic, standard, high 중 선택
LT_MIN_CONFIDENCE=50   # 언어 감지 최소 신뢰도 (0-100)
```

- **basic**: 가장 빠름, 단일 번역
- **standard**: 속도와 품질의 균형
- **high**: 최고 품질, 신뢰도 점수 포함

설정 변경 후 서비스 재시작:
```bash
docker-compose down
docker-compose up -d
```

## API 사용 예제

### 기본 번역

```bash
curl -X POST http://localhost:3006/translate \
  -H "Content-Type: application/json" \
  -d '{
    "q": "Hello, world!",
    "source": "en",
    "target": "ko"
  }'
```

### 언어 자동 감지

```bash
curl -X POST http://localhost:3006/translate \
  -H "Content-Type: application/json" \
  -d '{
    "q": "Hello, world!",
    "source": "auto",
    "target": "ko"
  }'
```

### 대체 번역 받기

```bash
curl -X POST http://localhost:3006/translate \
  -H "Content-Type: application/json" \
  -d '{
    "q": "Hello, world!",
    "source": "en",
    "target": "ko",
    "alternatives": 3
  }'
```

### 언어 감지

```bash
curl -X POST http://localhost:3006/detect \
  -H "Content-Type: application/json" \
  -d '{
    "q": "안녕하세요"
  }'
```

## 문제 해결

### 서비스가 시작되지 않을 때
```bash
# 3006 포트가 사용 중인지 확인
lsof -i :3006

# 상세 로그 확인
docker-compose logs libretranslate
```

### 모델 다운로드가 안 될 때
```bash
# 기존 볼륨 삭제 후 재시작
docker-compose down -v
docker-compose up -d

# 다운로드 진행 상황 모니터링
docker-compose logs -f libretranslate
```

### 메모리 부족 문제
메모리 문제가 발생하면:
1. `docker-compose.yml`에서 로드할 언어 수를 줄이기
2. 품질 레벨을 `standard` 또는 `basic`으로 변경

## 로컬 개발 환경 설정 (Python 직접 사용)

Docker 없이 로컬에서 개발하는 경우에만 필요합니다:

### 필요 사항
- Python 3.8 이상
- pip

### 설치 및 실행

```bash
# Python 버전 확인
python3 --version

# 가상 환경 생성
python3 -m venv venv

# 가상 환경 활성화
source venv/bin/activate  # Windows: venv\Scripts\activate

# 의존성 설치
pip install -e .

# 서비스 실행
libretranslate --host 0.0.0.0 --port 3006 --load-only en,ko,es,ja,zh
```

**주의**:
- 로컬 개발 환경에서는 코드 변경사항이 적용되지만, `beautifulsoup4` 등의 새로운 의존성은 `pyproject.toml`에 정의되어 있으므로 `pip install -e .` 명령으로 자동 설치됩니다.
- 언어 모델은 처음 실행 시 자동으로 다운로드됩니다 (~1-2GB).

## 라이센스

[GNU Affero General Public License v3](https://www.gnu.org/licenses/agpl-3.0.en.html)
