# 관리자 - 영웅/펫 관리 API

## 기본 정보

| 항목 | 내용 |
|------|------|
| Base URL | `/admin` |
| 인증 | 관리자 인증 쿠키 필요 |

---

## 영웅 관리

### 1. 영웅 목록 조회

| 항목 | 내용 |
|------|------|
| URL | `GET /admin/heroes` |
| Parameters | `query` (선택) - 이름 검색 (prefix match) |

#### Response

```json
{
    "code": "OK",
    "data": [
        {
            "id": 1,
            "name": "유신",
            "imageUrl": "https://r2.dev/heroes/유신.webp"
        }
    ]
}
```

---

### 2. 영웅 등록

| 항목 | 내용 |
|------|------|
| URL | `POST /admin/heroes` |
| Content-Type | `multipart/form-data` |

#### Parameters (form-data)

| 이름 | 타입 | 필수 | 설명 |
|------|------|----|------|
| name | String | O  | 영웅 이름 (중복 불가) |
| image | File | X  | 이미지 파일 (R2 업로드) |

#### Response

```json
{
    "code": "OK",
    "data": {
        "id": 9,
        "name": "새영웅",
        "imageUrl": "https://r2.dev/heroes/새영웅.webp"
    }
}
```

#### Error

| code | 설명 |
|------|------|
| `HERO_ALREADY_EXISTS` | 이미 존재하는 영웅 이름 |

---

### 3. 영웅 수정

| 항목 | 내용 |
|------|------|
| URL | `PUT /admin/heroes/{id}` |
| Content-Type | `multipart/form-data` |

#### Parameters (form-data)

| 이름 | 타입 | 필수 | 설명 |
|------|------|----|------|
| name | String | O  | 변경할 이름 |
| image | File | X  | 변경할 이미지 (미전송 시 기존 유지) |

#### Error

| code | 설명 |
|------|------|
| `HERO_NOT_FOUND` | 존재하지 않는 영웅 |
| `HERO_ALREADY_EXISTS` | 중복된 이름 |

---

### 4. 영웅 삭제

| 항목 | 내용 |
|------|------|
| URL | `DELETE /admin/heroes/{id}` |

#### Response

```json
{
    "code": "OK"
}
```

#### Error

| code | 설명 |
|------|------|
| `HERO_NOT_FOUND` | 존재하지 않는 영웅 |

---

## 펫 관리

### 5. 펫 목록 조회

| 항목 | 내용 |
|------|------|
| URL | `GET /admin/pets` |
| Parameters | `query` (선택) - 이름 검색 (prefix match) |

#### Response

```json
{
    "code": "OK",
    "data": [
        {
            "id": 1,
            "name": "멜키르",
            "imageUrl": null
        }
    ]
}
```

---

### 6. 펫 등록

| 항목 | 내용 |
|------|------|
| URL | `POST /admin/pets` |
| Content-Type | `multipart/form-data` |

#### Parameters (form-data)

| 이름 | 타입 | 필수 | 설명 |
|------|------|----|------|
| name | String | O  | 펫 이름 (중복 불가) |
| image | File | X  | 이미지 파일 |

#### Error

| code | 설명 |
|------|------|
| `PET_ALREADY_EXISTS` | 이미 존재하는 펫 이름 |

---

### 7. 펫 수정

| 항목 | 내용 |
|------|------|
| URL | `PUT /admin/pets/{id}` |
| Content-Type | `multipart/form-data` |

#### Parameters (form-data)

| 이름 | 타입 | 필수 | 설명 |
|------|------|----|------|
| name | String | O  | 변경할 이름 |
| image | File | X  | 변경할 이미지 |

#### Error

| code | 설명 |
|------|------|
| `PET_NOT_FOUND` | 존재하지 않는 펫 |
| `PET_ALREADY_EXISTS` | 중복된 이름 |

---

### 8. 펫 삭제

| 항목 | 내용 |
|------|------|
| URL | `DELETE /admin/pets/{id}` |

#### Error

| code | 설명 |
|------|------|
| `PET_NOT_FOUND` | 존재하지 않는 펫 |
