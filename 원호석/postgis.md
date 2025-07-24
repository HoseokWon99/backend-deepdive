# PostGIS란?

**PostGIS**는 PostgreSQL 관계형 데이터베이스에 **공간 데이터(geospatial data)**를 저장하고 질의할 수 있는 기능을 추가해주는 **오픈소스 확장(extension)**입니다. GIS(Geographic Information Systems) 기능을 지원하여 공간 정보 처리를 가능하게 합니다.

## 주요 기능

- 점(Point), 선(LineString), 다각형(Polygon) 등 **공간 데이터 타입** 지원
- 공간 인덱스 생성 (GiST, SP-GiST)
- 공간 연산자 및 함수 지원 (거리, 교차 여부, 포함 여부 등)
- GeoJSON, KML, GML 등 다양한 **지리 데이터 포맷과의 호환성**
- OpenStreetMap, QGIS, Leaflet, GeoServer 등과의 연동

---

## 설치 방법

### 1. PostgreSQL 설치

Ubuntu:
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### 2. PostGIS 설치

Ubuntu:
```bash
sudo apt install postgis postgresql-15-postgis-3
```

Mac (Homebrew):
```bash
brew install postgis
```

---

## 데이터베이스에 PostGIS 활성화

```sql
-- 새로운 데이터베이스 생성
CREATE DATABASE gisdb;

-- PostGIS 확장 활성화
\c gisdb
CREATE EXTENSION postgis;
```

---

## 주요 데이터 타입

| 타입         | 설명                           |
|--------------|--------------------------------|
| `geometry`   | 가장 일반적인 공간 데이터 타입  |
| `geography`  | 지구 곡률을 반영한 공간 타입   |
| `raster`     | 래스터 이미지 (DEM, 위성사진 등) |

---

## 주요 함수 및 연산자

| 함수/연산자          | 설명                                    |
|----------------------|-----------------------------------------|
| `ST_Distance(a, b)`  | 두 지오메트리 객체 사이의 거리 계산    |
| `ST_Intersects(a, b)`| 두 객체가 교차하는지 여부               |
| `ST_Within(a, b)`    | a가 b 내부에 포함되는지 여부            |
| `ST_Contains(a, b)`  | a가 b를 포함하는지 여부                 |
| `ST_AsGeoJSON(geom)` | GeoJSON 포맷으로 변환                   |
| `ST_Transform(geom, srid)` | 좌표계 변환 (예: WGS84 <-> UTM)    |

---

## GeoJSON이란?

**GeoJSON**은 공간 데이터를 표현하기 위한 JSON 기반의 형식입니다. 주로 웹 애플리케이션에서 지도 시각화를 위해 사용됩니다.

예시:
```json
{
  "type": "Point",
  "coordinates": [127.1, 37.5]
}
```

PostGIS 변환 함수:
```sql
SELECT ST_AsGeoJSON(geom) FROM places;
```

---

## WKT (Well-Known Text)란?

**WKT**는 공간 객체를 텍스트로 표현하는 포맷으로, 사람이 읽기 쉬운 형태입니다. PostGIS에서 공간 데이터를 입력할 때 자주 사용됩니다.

예시:
```sql
POINT(127.1 37.5)
LINESTRING(127.1 37.5, 127.2 37.6)
POLYGON((127.1 37.5, 127.2 37.6, 127.3 37.5, 127.1 37.5))
```

PostGIS 사용 예:
```sql
SELECT ST_GeomFromText('POINT(127.1 37.5)', 4326);
```

---

## 예제

```sql
-- POINT 생성
SELECT ST_GeomFromText('POINT(127.1 37.5)', 4326);

-- 두 위치의 거리 구하기
SELECT ST_Distance(
  ST_GeomFromText('POINT(127.1 37.5)', 4326),
  ST_GeomFromText('POINT(127.2 37.6)', 4326)
);

-- 특정 위치 반경 1km 이내 검색
SELECT name
FROM places
WHERE ST_DWithin(
  geom::geography,
  ST_MakePoint(127.1, 37.5)::geography,
  1000
);
```

---

## 공간 인덱스

PostGIS는 성능 향상을 위해 GiST (Generalized Search Tree) 인덱스를 사용합니다.

```sql
CREATE INDEX idx_places_geom
ON places
USING GIST (geom);
```

---

## 좌표계 (SRID)

- **SRID**: Spatial Reference System Identifier
- WGS84 (GPS 좌표계): `4326`
- UTM-K (한국 정밀측량): `5179`

좌표계 변환 예시:
```sql
SELECT ST_Transform(geom, 5179) FROM locations;
```

---

## 관련 도구 및 생태계

- **QGIS**: 데스크탑 GIS 소프트웨어 (PostGIS 직접 연동 가능)
- **GeoServer**: Web GIS 서버, PostGIS 지원
- **Leaflet / OpenLayers**: Web Mapping 라이브러리
- **pgRouting**: 경로탐색, 네트워크 분석 확장

---

## Redis GEO 기능

**Redis GEO**는 Redis에서 제공하는 공간 좌표 기반 명령어로, 위도와 경도 정보를 저장하고 이를 기반으로 검색, 거리 계산 등을 수행할 수 있습니다.

### 주요 명령어

| 명령어            | 설명 |
|-------------------|------|
| `GEOADD`          | 위치 정보(경도, 위도, 이름)를 추가 |
| `GEODIST`         | 두 지점 사이의 거리 계산 |
| `GEORADIUS`       | 지정한 반경 내 위치 검색 (이전 버전) |
| `GEOSEARCH`       | 기준점 또는 객체 기반 반경 검색 (추천) |
| `GEOPOS`          | 저장된 위치의 좌표 조회 |
| `GEOHASH`         | 지정된 위치의 geohash 문자열 조회 |

### 예제

```bash
# 위치 추가 (경도, 위도 순서)
GEOADD korea 126.9780 37.5665 "Seoul"
GEOADD korea 129.0756 35.1796 "Busan"

# 두 도시 간 거리 (단위: km)
GEODIST korea Seoul Busan km

# 반경 100km 내 도시 조회
GEOSEARCH korea FROMLONLAT 126.9780 37.5665 BYRADIUS 100 km WITHDIST
```

### 특징

- 매우 빠른 반경 검색 (O(log(N)) 성능)
- 단순한 위치 기반 서비스 구현에 적합
- SRID 없이 WGS84 (EPSG:4326) 좌표계를 기본 사용
- 복잡한 공간 연산 (겹침, 다각형 등)은 지원하지 않음 (PostGIS를 사용하는 것이 적합)

### 활용 예시

- 근처 매장 검색
- 배달 거리 계산
- 사용자 위치 기반 푸시 알림

---

## 참고 자료

- [PostGIS 공식 문서](https://postgis.net/documentation/)
- [PostgreSQL 공식 사이트](https://www.postgresql.org/)
- [QGIS 프로젝트](https://qgis.org/)
- [PostGIS 튜토리얼 (한국어)](https://postgis.net/workshops/postgis-intro/)
- [Redis GEO 명령어 문서](https://redis.io/docs/latest/commands/#geo)
