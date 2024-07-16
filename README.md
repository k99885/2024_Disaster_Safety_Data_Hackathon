# 2024 재난안전데이터 해커톤

- **기간**: 2024.06.11~2024.06.12 (무박 2일)
- **역할**: 팀 리더 및 데이터 분석
- **프로젝트 개요**: 재난데이터플랫폼의 데이터를 활용하여 의료사각지대 판별 및 해소 방안
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

```
file_path = "/content/data_1.csv"
data = pd.read_csv(file_path)

extracted_data_1 = data[['YR', 'DONG_CD', 'STATS_VL']]
filtered_data_1_2008 = extracted_data_1[extracted_data_1['YR'] == 2008]

output_file_path = '/content/extracted_data_1_2008.csv'
filtered_data_1_2008.to_csv(output_file_path, index=False)
```

의료사각지대 판별을 위하여 2008년도의 인구수,지역 코드를 추출하여 extracted_data_1_2008.csv에 저장하였습니다. 

이와같은 방법으로 지역별 병원수 데이터도 추출하여  extracted_data_3_2008.csv에 저장하였습니다.

추가적으로 지역별 고령화 인구수데이터,병원수 데이터를 하나의 csv로 합쳐 filtered_data_gyeongsangnamdo.csv에 저장 하였습니다.

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

마찬가지로 지역별 병원수의 데이터도 시각화 하였습니다.




