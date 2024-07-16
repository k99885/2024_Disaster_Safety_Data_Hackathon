# 2024 재난안전데이터 해커톤

- **기간**: 2024.06.11~2024.06.12 (무박 2일)
- **역할**: 팀 리더 및 데이터 분석
- **프로젝트 개요**: 재난데이터플랫폼의 데이터를 활용한 의료사각지대 판별 및 해소 방안
- **주요 활동 및 성과**:
  - **데이터 분석**: 재난데이터플랫폼에서 제공하는 다양한 데이터를 수집하여 의료 사각지대 분석
  - **시나리오 개발**: 분석 결과를 바탕으로 의료 사각지대에 비대면 처방 시나리오 제시
  - **수익화 모델 제시**: 재난 예측 및 대응 시스템을 기반으로 한 B2G 비즈니스 모델을 통한 수익 창출 방안 제시
## 사용기술
 - **프로그래밍 언어**: python
 - **데이터 분석 라이브러리**: pandas,folium

## 1. 데이터 수집
https://www.safetydata.go.kr

재난안전데이터공유플랫폼에서 고령인구수,병원수 데이터를 수집하였습니다.

<img width="394" alt="스크린샷 2024-07-16 오후 1 45 13" src="https://github.com/user-attachments/assets/0e8cfe2e-3e41-4c40-b2e5-755fa1edee11">

<img width="480" alt="스크린샷 2024-07-16 오후 1 49 07" src="https://github.com/user-attachments/assets/424d8cd4-22a4-431f-9894-ed1b1b2e50ea">

```
import requests
import urllib3
import csv

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

url = "https://www.safetydata.go.kr"
dataName = "/V2/api/DSSP-IF-00503"
serviceKey = "P7AD52K4K42A5009"

payloads = {
    "serviceKey": serviceKey,
    "returnType": "JSON",  # Specify return type as JSON
    "pageNo": "1",
    "numOfRows": "1000",
}

response_1 = requests.get(url + dataName, params=payloads, verify=False)

if response_1.status_code == 200:
    data = response_1.json()  
    records = data.get("body", [])
    if records:
        headers = records[0].keys()
        with open("data_1.csv", "w", encoding="utf-8", newline='') as file:
            writer = csv.DictWriter(file, fieldnames=headers)
            writer.writeheader()
            writer.writerows(records)
```

오픈 API를 이용하여 고령인구수 데이터를 data_1.csv에 저장하였습니다.

<img width="1311" alt="스크린샷 2024-07-16 오후 2 54 48" src="https://github.com/user-attachments/assets/cf812b81-76f9-4ec1-81fe-fc33ba453334">

```
file_path = "/content/data_1.csv"
data = pd.read_csv(file_path)

extracted_data_1 = data[['YR', 'DONG_CD', 'STATS_VL']]
filtered_data_1_2008 = extracted_data_1[extracted_data_1['YR'] == 2008]

output_file_path = '/content/extracted_data_1_2008.csv'
filtered_data_1_2008.to_csv(output_file_path, index=False)
```

의료사각지대 판별을 위하여 2008년도의 인구수,지역 코드를 추출하여 extracted_data_1_2008.csv에 저장하였습니다. 

<img width="172" alt="스크린샷 2024-07-16 오후 2 55 12" src="https://github.com/user-attachments/assets/5e173f48-d4ed-450e-88ef-91f2af1db246">

이와같은 방법으로 지역별 병원수 데이터도 추출하여  extracted_data_3_2008.csv에 저장하였습니다.

<img width="181" alt="스크린샷 2024-07-16 오후 2 55 33" src="https://github.com/user-attachments/assets/1f1db635-7e3d-4732-8a16-5e95686058ed">

추가적으로 지역별 고령화 인구수데이터,병원수 데이터를 하나의 csv로 합쳐 filtered_data_gyeongsangnamdo.csv에 저장 하였습니다.

<img width="467" alt="스크린샷 2024-07-16 오후 2 56 19" src="https://github.com/user-attachments/assets/e212d35c-b01d-465d-89d5-12c7d5864c71">

## 2. 데이터 분석 

수집한 데이터를 분석하기 위하여 FOLIUM을 사용하여 데이터 시각화 과정을 진행하였습니다.
```
def get_color(feature):
    sgg_nm = feature['properties']['name']
    if sgg_nm in data['SGG_NM'].unique():
        stats_vl = data[data['SGG_NM'] == sgg_nm]['STATS_VL'].sum()
        if stats_vl > 300000:
            return '#800026'
        elif stats_vl > 20000:
            return '#BD0026'
        elif stats_vl > 16000:
            return '#E31A1C'
        elif stats_vl > 13000:
            return '#FC4E2A'
        elif stats_vl > 10000:
            return '#FD8D3C'
        else:
            return '#FEB24C'
    else:
        return '#FFFFFF'  # 데이터가 없는 경우 흰색
```

지역별로 고령화 인구수가 많을수록 진한 색으로 설정하였습니다.


```
# 지도 생성
m = folium.Map(location=[35.1796, 128.1113], zoom_start=9)

# GeoJSON 데이터를 지도에 추가
folium.GeoJson(
    geo_data,
    style_function=lambda feature: {
        'fillColor': get_color(feature),
        'color': 'black',
        'weight': 2,
        'fillOpacity': 0.7,
    }
).add_to(m)

# 지도 저장
map_path = '/content/gyeongsangnamdo_stats_vl_regions.html'
m.save(map_path)
```
<img width="643" alt="스크린샷 2024-06-12 오전 4 09 32" src="https://github.com/user-attachments/assets/06d82a19-fe31-4c3e-a94e-43b74afa73e0">

```
data['STATS_VL_DIVIDED'] = data['STATS_VL'] / data['STATS_VL_2']
```
각 지역별로 고령인구수/병원수 값들을 새로운 열에 추가하였습니다. 

<img width="597" alt="스크린샷 2024-07-16 오후 3 22 39" src="https://github.com/user-attachments/assets/83cd7ca8-1630-41ea-9ffa-eca2b2bb534e">



-> 이때 각 지역별로 고령인구수/병원수의 값(STATS_VL_DIVIDED)이 다른 지역에 비하여 10배가 넘는 지표가 나타나 이러한 지역을 의료사각지대로 판별 하였습니다.

<img width="632" alt="스크린샷 2024-06-12 오전 4 09 01" src="https://github.com/user-attachments/assets/2a42046a-3081-4e66-b909-966c48ee37ef">

-> 데이터 분석을 통하여 의료 사각지대로 판별된 지역입니다. 

## 3. BM 과 수익 창출 방안
의료 사각지대를 해소하기 위해 데이터 분석을 통해 판별된 지역에 대해 다음과 같은 비즈니스 모델(BM)을 제시하였습니다. 또한 B2G(Business to Government)를 통한 수익 창출 방안도 함께 제안하였습니다.

- **비즈니스 모델(BM)**:
  - **의약품 선입선출 시스템**: 의료 접근성이 낮은 지역에 의약품 관리 시스템을 도입하여 의약품을 효율적으로 관리하고 배급.
  - **비대면 처방 시스템**: 인터넷과 모바일 기술을 활용하여 원격으로 진료 및 처방을 제공, 환자가 편리하게 약을 처방 받을 수 있도록 함.
  - **의약품 전달 서비스**: 의약품을 직접 환자들에게 배송하는 서비스를 제공하여 접근성이 낮은 지역에서도 쉽게 약을 받을 수 있도록 함.
  - 
- **B2G를 통한 수익 창출 방안**:

  - **데이터 제공 및 분석 서비스**:정부 기관에 의료 데이터 분석 서비스를 제공하여 정책 수립에 기여하고, 이에 따른 수익 창출.
  - **정부 보조금 및 지원금**:정부의 의료 사각지대 해소 정책에 따라 보조금 및 지원금을 확보.






