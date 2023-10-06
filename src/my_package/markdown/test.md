# 새만금 위젯(API) 개발 현황

새만금 위젯(API) 개발 현황

## 차량 간선 배차 현황 위젯
- 물류센터 화물 차량 입/출고 알림 위젯
    
    >* 입출고type: 입고
    >* 입출고시간: "2022-11-01T05:46:39"
    >* 차량ID: KMFGA13TPCC209831
    >* 냉동차type: 상온
    >* 차량번호: 80가 1234
    >* 차량모델명: 현대 쏘렌토
    >* 차량톤수: 5
    >* 출발지: 이천
    >* 출발날짜: "2022-11-01T05:46:39"
    >* 도착지: 포항
    >* 도착날짜: "2022-11-01T11:37:29"
    >* 배송type: 경유
    
    
- 코드

```python
import random
import datetime
import pandas as pd

from call_mongo import MyAPI  

myDB = MyAPI()

Operate_Start = 15
Operate_End   = 26

city_list = ['서울','대구', '부산', '인천', '광주', '울산', '성남', '화성', '아산','광명',
            '천안', '전주', '안산', '김해', '평택', '군포', '군산', '여수', '순천','경주']
delivery_type = ["경유", "편도"]

def create_collection():
    coll_radiusalter = myDB[""]
    return coll_radiusalter

def get_cars_info():
    collection_name = "carinfo"
    coll = myDB.mydb.get_collection(collection_name)
    cars_info = coll.find({"조직":"CompanyA"})
    return cars_info

def create_time(today, min_h, max_h):
    h = random.randint(min_h, max_h)
    m = random.randint(0, 59)
    s = random.randint(0, 59)
    if h > 23:
        h -= 24
        create_time = today.replace(hour=h, minute=m, second=s)
        create_time = create_time.replace(day=create_time.day+1)
    else:
        create_time = today.replace(hour=h, minute=m, second=s)

    return create_time

def create_data(set_Day, car_info):
    type = random.randint(0,1)                                          # 0: 입고, 1: 출고
    out_time = None
    data_time = None
    if type == 0:
        # 입고시간
        enter_time  = create_time(set_Day, Operate_Start, 17)            # 15시~18시 전 까지 차량 물류센터 입고
        arrive_time = create_time(set_Day, 20, 26)                       # 20시부터 익일 3시 전까지 도착완료 
        enter_time  = enter_time.strftime('%Y-%m-%dT%H:%M:%S')
        data_time   = enter_time                                         # 데이터 생성 시간 = 입고시간
    else: 
        # 출고시간
        enter_time  = create_time(set_Day, Operate_Start, 17)                  
        out_time    = create_time(set_Day, enter_time.hour, 19)         # 입고시간 후 부터 20시 전까지 물류센터 출고         
        arrive_time = create_time(set_Day, out_time.hour, 26)           # 출고시간 후 부터 익일 3시 전까지 도착완료 
        out_time    = out_time.strftime('%Y-%m-%dT%H:%M:%S')
        data_time   = out_time                                          # 데이터 생성 시간 = 출고시간

    departure  = city_list[random.randint(0,len(city_list)-1)]          # 출발지
    arrive     = departure
    while departure == arrive:
        arrive = city_list[random.randint(0,len(city_list)-1)]          # 도착지
    arrive_time = arrive_time.strftime('%Y-%m-%dT%H:%M:%S')             # 도착날짜
    
    delivery   = delivery_type[random.randint(0,1)]                     # 배송Type
    type       = "입고" if type == 0 else "출고"                         # 입출고Type

    return [data_time, type, enter_time, out_time, car_info["ID(차대번호)"], car_info["차량타입"], car_info["차량번호"], car_info["모델명"], 
            car_info["차량톤수"], departure, out_time, arrive, arrive_time, delivery]

def main():
    cars_info   = get_cars_info()
    # collection  = create_collection()
    today     = datetime.datetime.now()
    create_datas = []
    for info in cars_info:
        set_day   = today.replace(day=today.day-1)
        virtual_data = create_data(set_day, info)
        create_datas.append(virtual_data)

    cols = ["데이터생성시간", "입출고타입", "입고시간", "출고시간", "차량ID", "냉동차Type", "차량번호", "차량모델명", "차량톤수", "출발지", "출발날짜", "도착지", "도착날짜", "배송Type"  ]
    df = pd.DataFrame(create_datas, columns=cols)
    print(df)

if __name__ == '__main__':
    main()
```