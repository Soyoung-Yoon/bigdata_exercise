# PIL / Pillow — Image 학습 콘텐츠 정리

> **파일명**: `pil_quiz.html`  
> **목적**: CNN 학습 기초, 실기 시험 대비 완전 암기  
> **구성**: 플래시카드 · 빈칸 퀴즈 · 치트시트 · 오답노트 (4탭)

---

## 1. 학습 범위 및 목표

```
from PIL import Image
```

| 목표 | 내용 |
|---|---|
| 핵심 메서드 8개 완전 암기 | open, show, save, resize, crop, rotate, convert, transpose |
| CNN 전처리 흐름 이해 | open → convert → resize → save 패턴 |
| 실습 코드 패턴 암기 | for 루프 전처리, 데이터 증강 패턴 |
| 실기 시험 빈칸 대비 | 메서드명 + 파라미터 형식까지 |

---

## 2. 메서드 8개 상세 정리

### 2-1. Image.open(path)
- **분류**: PIL.Image 모듈 함수
- **역할**: 이미지 파일을 열어 PIL Image 객체로 반환
- **파라미터**: `path: str` — 이미지 파일 경로
- **반환값**: `PIL.Image.Image` 객체
- **지원 형식**: JPEG, PNG, BMP, GIF, TIFF 등
- **CNN 맥락**: 가장 먼저 호출. `img.size`로 원본 크기 확인

```python
from PIL import Image

img = Image.open("daisy.jpg")
print(img.size)   # (width, height) 튜플
print(img.mode)   # 'RGB', 'L', 'RGBA' 등
```

---

### 2-2. img.show()
- **분류**: PIL Image 인스턴스 메서드
- **역할**: OS 기본 이미지 뷰어로 즉시 열어 확인
- **파라미터**: 없음
- **반환값**: `None`
- **CNN 맥락**: 전처리 전/후 상태를 빠르게 육안 확인
- **주의**: Jupyter에서는 `plt.imshow(img)` + `plt.show()` 권장

```python
img = Image.open("daisy.jpg")
img.show()  # OS 기본 뷰어

# Jupyter 환경 권장 방식
import matplotlib.pyplot as plt
plt.imshow(img)
plt.show()
```

---

### 2-3. img.save(path)
- **분류**: PIL Image 인스턴스 메서드
- **역할**: 이미지를 지정한 경로에 저장
- **파라미터**: `path: str` — 저장 경로 (확장자 포함)
- **반환값**: `None`
- **형식 결정**: 확장자 자동 인식 (.jpg → JPEG, .png → PNG)
- **CNN 맥락**: 전처리 완료된 이미지를 학습용 데이터셋으로 저장

```python
img.save("output.jpg")            # JPEG 저장
img.save("output.png")            # PNG 저장
img.save("out.jpg", quality=95)   # 품질 옵션 (JPEG 전용)
```

---

### 2-4. img.resize((w, h))
- **분류**: PIL Image 인스턴스 메서드
- **역할**: 이미지를 지정한 픽셀 크기로 변환
- **파라미터**: `(width, height): tuple` — 목표 크기 (픽셀)
- **반환값**: 새로운 `PIL.Image.Image` 객체 (원본 유지)
- **주의**: 비율 자동 유지 안 됨. 인자가 **튜플** (이중 괄호 주의!)
- **CNN 맥락**: CNN은 고정 크기 입력 요구. VGG/ResNet = 224×224, 소형 = 32×32 또는 64×64

```python
img_resized = img.resize((224, 224))  # CNN 표준 입력
img_resized = img.resize((64, 64))    # 소형 CNN

# for 루프 전처리 패턴 (실기 자주 출제)
for f in file_list:
    img = Image.open(path + f).resize((224, 224))
    plt.imshow(img)
    plt.show()
```

> ⚠️ **이중 괄호 실수 주의**: `.resize(224, 224)` ❌ → `.resize((224, 224))` ✅

---

### 2-5. img.crop((l, t, r, b))
- **분류**: PIL Image 인스턴스 메서드
- **역할**: (left, top, right, bottom) 좌표로 직사각형 영역을 잘라냄
- **파라미터**: `(left, top, right, bottom): tuple` — 잘라낼 영역 좌표 (픽셀)
- **반환값**: 새로운 `PIL.Image.Image` 객체
- **CNN 맥락**: 데이터 증강(Data Augmentation) — 랜덤 크롭으로 학습 데이터 다양화

```python
# 고정 영역 자르기
region = img.crop((0, 0, 100, 100))

# 중앙 영역 자르기
w, h = img.size
box = (w//4, h//4, w*3//4, h*3//4)
center = img.crop(box)
```

> 💡 **좌표 순서 암기**: Left · Top · Right · Bottom → **"좌상우하"**

---

### 2-6. img.rotate(angle)
- **분류**: PIL Image 인스턴스 메서드
- **역할**: 이미지를 반시계 방향으로 angle도 회전
- **파라미터**:
  - `angle: float` — 회전 각도 (반시계 방향 기준)
  - `expand: bool` — 캔버스 자동 확장 여부 (기본 `False`)
- **반환값**: 새로운 `PIL.Image.Image` 객체
- **CNN 맥락**: 데이터 증강 필수. 회전 불변성(rotation invariance) 학습

```python
rotated = img.rotate(90)             # 반시계 90도
rotated = img.rotate(45, expand=True) # 확장해서 자르지 않음
rotated = img.rotate(-90)            # 시계 방향 90도 (음수)
```

> 💡 **방향 암기**: `rotate(90)` = 반시계. 시계 방향은 음수(-90) 또는 `transpose(ROTATE_90)`

---

### 2-7. img.convert(mode)
- **분류**: PIL Image 인스턴스 메서드
- **역할**: 이미지의 색상 모드를 변환
- **파라미터**: `mode: str`
  - `'L'` — 흑백(그레이스케일, 1채널)
  - `'RGB'` — 컬러(3채널)
  - `'RGBA'` — 투명도 포함(4채널)
- **반환값**: 새로운 `PIL.Image.Image` 객체
- **CNN 맥락**: CNN 입력 채널 수와 반드시 일치. 컬러 모델 = RGB, 흑백 모델 = L

```python
gray = img.convert('L')    # 흑백 변환 (1채널)
rgb  = img.convert('RGB')  # 컬러 보장 (3채널)

# PNG(RGBA) → JPEG 저장 전 필수 변환
img.convert('RGB').save('out.jpg')
```

> 💡 **L은 Luminance(밝기)의 약자**

---

### 2-8. img.transpose(method)
- **분류**: PIL Image 인스턴스 메서드
- **역할**: 대칭 변환 또는 회전
- **파라미터**: `method` — Image 모듈의 상수
  - `Image.FLIP_LEFT_RIGHT` — 좌우 반전
  - `Image.FLIP_TOP_BOTTOM` — 상하 반전
  - `Image.ROTATE_90` / `ROTATE_180` / `ROTATE_270` — 회전
- **반환값**: 새로운 `PIL.Image.Image` 객체
- **CNN 맥락**: 좌우 반전이 가장 흔한 데이터 증강. 학습 데이터를 2배로 늘리는 효과

```python
flipped_lr = img.transpose(Image.FLIP_LEFT_RIGHT)   # 좌우 대칭
flipped_tb = img.transpose(Image.FLIP_TOP_BOTTOM)   # 상하 대칭
rot90      = img.transpose(Image.ROTATE_90)         # 시계방향 90도
```

---

## 3. CNN 전처리 핵심 패턴 (실기 자주 출제)

### 패턴 1: 기본 전처리 파이프라인
```python
img = Image.open("input.png").convert('RGB').resize((224, 224))
img.save("processed.jpg")
```

### 패턴 2: for 루프 배치 전처리
```python
for img_file in daisy_file[:2]:
    img = Image.open(daisy_path + img_file).resize((224, 224))
    plt.title(img_file + ' : daisy')
    plt.imshow(img)
    plt.show()
```

### 패턴 3: 배치 저장
```python
import os
for fname in os.listdir("./data/"):
    img = Image.open("./data/" + fname).convert('RGB').resize((64, 64))
    img.save("./output/" + fname)
```

### 패턴 4: 데이터 증강 조합
```python
img = Image.open("cat.jpg")
augmented = [
    img.rotate(90),
    img.transpose(Image.FLIP_LEFT_RIGHT),
    img.rotate(-90),
    img.transpose(Image.FLIP_TOP_BOTTOM),
]
```

---

## 4. 플래시카드 목록 (8장)

| # | 앞면 (질문) | 뒷면 (정답 핵심) | 카테고리 |
|---|---|---|---|
| 1 | `Image.open(path)`는 무엇을 반환? | `PIL.Image.Image` 객체 | open |
| 2 | `img.show()`의 반환값은? | `None` (OS 뷰어로 열기) | show |
| 3 | `img.save(path)`에서 형식은 어떻게 결정? | 확장자 자동 인식 | save |
| 4 | `img.resize()`의 인자 형태는? | 튜플 `(w, h)` — 이중 괄호! | resize |
| 5 | `img.crop()`의 파라미터 순서는? | left, top, right, bottom | crop |
| 6 | `img.rotate(90)`은 어느 방향? | 반시계 방향 90도 | rotate |
| 7 | 흑백 변환: `img.convert(___)`? | `'L'` (Luminance) | convert |
| 8 | 좌우 반전 코드는? | `img.transpose(Image.FLIP_LEFT_RIGHT)` | transpose |

---

## 5. 빈칸 퀴즈 목록 (8문제)

| # | 태그 | 빈칸 정답 | 핵심 포인트 |
|---|---|---|---|
| Q1 | CNN 전처리 | `open`, `resize` | for 루프 + 224×224 패턴 |
| Q2 | 모드 변환 | `open`, `convert`, `save` | 흑백 변환 전체 흐름 |
| Q3 | 데이터 증강 | `transpose`, `FLIP_LEFT_RIGHT` | Image 상수 이름 |
| Q4 | 회전 | `rotate`, `90` | 반시계 방향 각도 |
| Q5 | 이미지 확인 | `open`, `show` | 기본 뷰어 열기 |
| Q6 | CNN 파이프라인 | `convert`, `resize`, `save` | open → convert → resize → save |
| Q7 | 크롭 | `crop` | (l, t, r, b) 순서 |
| Q8 | 배치 전처리 | `open`, `convert`, `resize`, `save` | 4가지 메서드 체이닝 |

---

## 6. 치트시트 전체 요약

| 메서드 | 반환값 | 파라미터 | CNN 용도 |
|---|---|---|---|
| `Image.open(path)` | PIL Image 객체 | path: str | 이미지 로드 |
| `img.show()` | None | 없음 | 육안 확인 |
| `img.save(path)` | None | path: str | 전처리 결과 저장 |
| `img.resize((w,h))` | 새 PIL Image | (width, height): tuple | 입력 크기 통일 |
| `img.crop((l,t,r,b))` | 새 PIL Image | 좌표 tuple | 랜덤 크롭 증강 |
| `img.rotate(angle)` | 새 PIL Image | angle: float, expand: bool | 회전 증강 |
| `img.convert(mode)` | 새 PIL Image | 'L' / 'RGB' / 'RGBA' | 채널 통일 |
| `img.transpose(method)` | 새 PIL Image | Image.FLIP_* / ROTATE_* | 반전·회전 증강 |

---

## 7. 자주 하는 실수 모음

| 실수 | 잘못된 코드 | 올바른 코드 |
|---|---|---|
| resize 괄호 누락 | `img.resize(224, 224)` | `img.resize((224, 224))` |
| convert 인자 소문자 | `img.convert('rgb')` | `img.convert('RGB')` |
| rotate 방향 혼동 | 양수 = 시계 방향? | 양수 = **반시계** 방향 |
| transpose 상수 소문자 | `Image.flip_left_right` | `Image.FLIP_LEFT_RIGHT` |
| PNG → JPEG 직접 저장 | `img.save('out.jpg')` | `img.convert('RGB').save('out.jpg')` |
| crop 순서 혼동 | `(top, left, bottom, right)` | `(left, top, right, bottom)` |

---

## 8. 추가 학습 예정 항목 (pil_quiz.html 확장용)

다음 항목들을 추가하면 더 풍부한 학습이 가능합니다.

| 항목 | 메서드 | 설명 |
|---|---|---|
| 이미지 정보 | `img.size`, `img.mode`, `img.format` | 속성(attribute) 확인 |
| numpy 변환 | `np.array(img)`, `Image.fromarray(arr)` | CNN 모델 입력 변환 |
| 썸네일 | `img.thumbnail((w, h))` | 비율 유지 리사이즈 |
| 붙여넣기 | `img.paste(img2, (x, y))` | 이미지 합성 |
| 필터 | `ImageFilter.BLUR`, `SHARPEN` | 이미지 필터 적용 |
| 그리기 | `ImageDraw.Draw(img)` | 이미지에 텍스트/도형 그리기 |

---

## 9. 다음 요청 시 제공할 정보 템플릿

```
[추가 학습 항목]
메서드명: img.thumbnail() 등
학습 목표: ...
실습 코드:
    (노트북 코드 붙여넣기)
CNN 연관성: ...
퀴즈 예시: (있으면)
난이도: 기초 / 중급 / 고급
플래시카드 필요: Y / N
치트시트 추가: Y / N
```
