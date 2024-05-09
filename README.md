# finalprojectAI
Check out groceries at the grocery store using recognition software
import cv2
import numpy as np
from keras.preprocessing.image import img_to_array
from keras.models import load_model
import pandas as pd

# Load mô hình nhận diện bánh từ file .h5
model = load_model('myenv/projectAI1.h5')

# Danh sách nhãn của các loại bánh
label_list = ['Fami','Milk','Oil','Omo','Soy','Sunlight','bun_an_lien','Chocopie','chocopie_box','cosi_cycle','cosi_oc_que_dua','cosi_oc_que_trung','dove','fish_sauce','lifebuoy','mi_cay','mi_hu','mi_tron','sua_hop','sugar','sunsik']

# Giá của từng loại bánh
prices = {
    'Fami': 20000,
    'Milk': 15000,
    'Oil': 25000,
    'Omo': 30000,
    'Soy': 18000,
    'Sunlight': 22000,
    'bun_an_lien': 12000,
    'Chocopie': 25000,
    'chocopie_box': 35000,
    'cosi_cycle': 18000,
    'cosi_oc_que_dua': 20000,
    'cosi_oc_que_trung': 20000,
    'dove': 35000,
    'fish_sauce': 30000,
    'lifebuoy': 25000,
    'mi_cay': 15000,
    'mi_hu': 12000,
    'mi_tron': 20000,
    'sua_hop': 30000,
    'sugar': 15000,
    'sunsik': 18000
}

# Biến toàn cục để lưu danh sách các món đồ và số lượng
items = []

# Hàm nhận diện bánh từ ảnh
def detect_cake(image):
    # Tiền xử lý ảnh
    image = cv2.resize(image, (224, 224))
    image = img_to_array(image)
    image = np.expand_dims(image, axis=0)

    # Dự đoán bánh từ ảnh
    result = model.predict(image)
    predicted_label_index = np.argmax(result)
    predicted_label = label_list[predicted_label_index]
    
    return predicted_label

# Hàm tính tổng số tiền dựa trên loại bánh và số lượng
def calculate_total_bill():
    total_bill = 0
    for item in items:
        cake_name, quantity = item
        if cake_name in prices:
            total_bill += prices[cake_name] * quantity
    return total_bill

# Hàm chụp ảnh từ camera và nhận diện bánh
def detect_and_calculate_bill():
    cap = cv2.VideoCapture(0)

    while True:
        ret, frame = cap.read()
        
        # Hiển thị frame lên màn hình
        cv2.imshow('Camera', frame)

        # Nhấn phím 'q' để kết thúc chương trình
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

        # Nhấn phím 's' để chụp ảnh và nhận diện bánh
        elif cv2.waitKey(1) & 0xFF == ord('s'):
            # Nhận diện món đồ từ frame
            detected_food = detect_cake(frame)
            print("Nhận diện bánh:", detected_food)
            quantity = int(input("Nhập số lượng bánh: "))
            items.append((detected_food, quantity))
            
            # Hiển thị lại frame để nhấn phím 's' hoặc 'q'
            cv2.imshow('Camera', frame)

        # Nhấn phím 'x' để kết thúc mua hàng và đóng camera
        elif cv2.waitKey(1) & 0xFF == ord('x'):
            break
        
    cap.release()
    cv2.destroyAllWindows()

    total_bill = calculate_total_bill()
    print("Tổng số tiền cần thanh toán:", total_bill)

    # Tạo DataFrame từ danh sách các mặt hàng
    df = pd.DataFrame(items, columns=['Loại Bánh', 'Số Lượng'])
    
    # Tính tổng tiền cho từng loại bánh
    df['Tổng Tiền'] = df.apply(lambda row: prices[row['Loại Bánh']] * row['Số Lượng'], axis=1)
    
    # Xuất DataFrame vào file Excel
    df.to_excel('bill.xlsx', index=False)
    print("Đã lưu thông tin vào file bill.xlsx")

# Gọi hàm để chụp ảnh từ camera và nhận diện bánh
detect_and_calculate_bill()
