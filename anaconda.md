# Anaconda 安裝

Anaconda是目前最受歡迎的Python數據科學平台，除了有眾多使用者及企業用戶外，目前也超過1000種的Data Science Packages可使用

適用於Windows、Linux和MacOS 不同作業系統環境下的conda軟件包(package)和虛擬環境管理器

對於在安裝、執行及升級複雜的數據科學(Data Science)及機器學習(Machine Learning)環境上變得簡單快速

----

#### Anaconda安裝時會出現一個bug，就是使用者名稱必須為英文，所以以下過程皆是在使用者名稱為英文前提下進行
#### 使用者名稱若不是英文，則必須在電腦上新增一個使用者再進行以下步驟

----

### Step 1 

- 進入 https://www.anaconda.com/products/distribution

- 到網頁最下方 選取適合的版本下載

![image](https://user-images.githubusercontent.com/101647060/176829479-96247b67-d761-4ce4-a0b1-9d2484a0ef33.png)


- 下載過程皆使用預設就好       
 
![image](https://user-images.githubusercontent.com/101647060/176830053-33ed13bd-c34b-45fa-9398-6998fa6c8812.png)

----

### Step 2 


- Anaconda安裝完之後，開啟Anaconda Prompt

![image](https://user-images.githubusercontent.com/101647060/176830431-e153bfc3-6ce1-4d41-9c4d-d120d797aace.png)
![image](https://user-images.githubusercontent.com/101647060/176830679-510e6259-e1a7-476c-ac84-96e6c43b14b6.png)

----

### Step 3  環境建置

- 輸入 conda create --name 環境名字 python=...

- 例如 : conda create --name earthscience python=3.9.12

- 想要確定python版本，只需要輸入python即會出現以下畫面(可知版本為3.9.12)

![image](https://user-images.githubusercontent.com/101647060/176831066-a6dc315a-afd1-4c3b-a661-d697d83eb225.png)

- 執行命令後 (conda create --name earthscience python=3.9.12)，會開始環境建置

![image](https://user-images.githubusercontent.com/101647060/176831354-2580ce95-1995-4362-afd2-f3e765fbf605.png) 途中跑出這個，按enter即可 (要確保在[y]上，左右鍵可調整)

- 出現以下畫面即是環境建置成功

![image](https://user-images.githubusercontent.com/101647060/176831414-306e73fb-7f3b-45f7-9ac3-6e1d382ddb8e.png)

----

### Step 4  模組下載

- 需先啟動環境 

- 輸入 conda activate 環境名字
- 例如 : conda activate earthscience

![image](https://user-images.githubusercontent.com/101647060/176834037-2b165cc7-8755-41e7-9835-e09ca24af85e.png)

- 命令執行成功，前面的 (base) 會變成 (環境名字)
- 環境啟動成功後，就可以安裝模組

- 輸入 conda install 模組名字
- 常用模組有 pandas、matplotlib、numpy

- pygmt 下載語法較特別
- 輸入 conda install pygmt --channel conda-forge

- bert 安裝python建議為3.7










