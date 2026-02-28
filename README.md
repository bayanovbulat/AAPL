ЛАБОРАТОРНАЯ РАБОТА № 1
Подготовка данных для анализа.
В работе используем язык программирования Python, пакеты данных sqlite3, numpy, matplotlib, pandas, scipy и т.д. Для установки Python воспользуемся: https://www.python.org/downloads/. Для установки пакетов данных в консоли (WindowsPowerShell) прописываем команду, например, pip install matplotlib.
Рассмотрим данные в csv с данными о ценах акций компании Apple Inc. Скачаем с сайта finam.ru, перейдя в: Котировки/Акции/США/Apple Inc./Экспорт. Установим даты 01.01.2023 – 01.01.2025 с периодичностью в одну неделю. В итоге получаем файл с названием AAPL_230101_250101.csv.
Исходные данные: 105 строк (105 недель в указанный период), 9 столбцов (идентификатор бумаги <TICKER>, временной интервал <PER>, дата <DATE>, время <TIME>, цена первой сделки <OPEN>, максимальная цена сделки <HIGH>, минимальная цена сделки <LOW>, цена последней сделки <CLOSE>, объем, общее количество акций, сменивших владельца, демонстрирует интерес к ценной бумаге <VOL>).
Выполним запись исходных данных: с csv файла в переменную типа list (Листинг программы №1.1), в файл базы данных sqlite3 (Листинг программы №1.2), с файла базы данных sqlite3 в переменную типа list (Листинг программы №1.3).
В папке dataanalysis создаем файл inputdata.py.
Листинг программы №1.1 «inputdata.py»
import pandas as pd
# Загружаем CSV-файл в DataFrame
df = pd.read_csv('AAPL_230101_250101.csv', delimiter=';')
# Преобразуем DataFrame в список списков (list of lists)
dataset = df.values.tolist()
# Выводим информацию для проверки
print(f"Тип переменной dataset: {type(dataset)}")
print(f"Количество строк в dataset: {len(dataset)}")
print(f"Количество столбцов в dataset: {len(dataset[0]) if dataset else 0}")
print("\nПервые 5 строк dataset:")
for row in dataset[:5]:
    print(row)
Теперь создадим файл inputdata.py для заполнения данными таблиц.
Листинг программы №1.2 «inputdata.py»
import pandas as pd
import sqlite3
# Загружаем CSV-файл в DataFrame
df = pd.read_csv('AAPL_230101_250101.csv', delimiter=';')
# Преобразуем DataFrame в список списков (list of lists)
dataset = df.values.tolist()
# Подключаемся к базе данных (файл будет создан автоматически)
conn = sqlite3.connect('database.db')
cursor = conn.cursor()
# Создаем таблицу (если она не существует)
cursor.execute('''
    CREATE TABLE IF NOT EXISTS weekly_prices (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        ticker TEXT NOT NULL,
        period TEXT NOT NULL,
        date INTEGER NOT NULL,
        time TEXT,
        open_price REAL,
        high_price REAL,
        low_price REAL,
        close_price REAL,
        volume INTEGER,
        UNIQUE(ticker, date)
    )
''')
for row in dataset:
    cursor.execute('''
        INSERT OR IGNORE INTO weekly_prices
             (ticker, period, date, time, open_price, high_price, low_price, close_price, volume) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
    ''', (row[0], row[1], int(row[2]), row[3], float(row[4]), float(row[5]), float(row[6]), float(row[7]), int(row[8])))
# Сохраняем изменения
conn.commit()
# Закрываем соединение
conn.close()

Теперь с файла database.db создадим SQL запрос типа SELECT.
Листинг программы №1.3 «inputdata.py»
import sqlite3
# Подключаемся к базе данных
conn = sqlite3.connect('database.db')
cursor = conn.cursor()
# Выполняем SELECT запрос для получения всех данных
cursor.execute('''
    SELECT ticker, period, date, time, open_price, high_price, low_price, close_price, volume
    FROM weekly_prices
    ORDER BY date
''')
# Загружаем все строки в переменную dataset (список списков)
dataset = cursor.fetchall()
# Закрываем соединение
conn.close()
# Проверяем результат
print(f"Тип переменной dataset: {type(dataset)}")
print(f"Количество записей: {len(dataset)}")
print(f"Количество столбцов: {len(dataset[0]) if dataset else 0}")
# Выводим первые 5 записей
print("\nПервые 5 записей из БД:")
for i, row in enumerate(dataset[:5]):
    print(f"{i+1}. {row}")
Примечание:
Для загрузки исходных данных можно воспользоваться файлами из репозитория https://github.com/bayanovbulat/AAPL, например:
df=pd.read_csv(https://raw.githubusercontent.com/bayanovbulat/AAPL/refs/heads/main/AAPL_230101_250101.csv', delimiter=';')
https://raw.githubusercontent.com/bayanovbulat/AAPL/refs/heads/main/:
# Исходные данные в период 230101_250101 с периодичностью в одну неделю

AAPL_230101_250101.csv, CSCO_230101_250101.csv, MSFT_230101_250101.csv, QCOM_230101_250101.csv, SAMSUNGCDGN_230101_250101.csv
# Исходные данные в период 230101_250101 с периодичностью в один день
AAPL_230101_250101_day.csv, CSCO_230101_250101_day.csv, MSFT_230101_250101_day.csv, QCOM_230101_250101_day.csv, SAMSUNGCDGN_230101_250101_day.csv
# Исходные данные в период 250101_260101 с периодичностью в одну неделю
AAPL_250101_260101.csv, CSCO_250101_260101.csv, MSFT_250101_260101.csv, SAMSUNGCDGN_250101_260101.csv
# Исходные данные в период 250101_260101 с периодичностью в один день
AAPL_250101_260101_day.csv, CSCO_250101_260101_day.csv, MSFT_250101_260101_day.csv, QCOM_250101_260101_day.csv, SAMSUNGCDGN_250101_260101_day.csv
Для загрузки исходных данных Google Trends можно воспользоваться файлами из репозитория https://github.com/bayanovbulat/AAPL, например:
df_google=pd.read_csv('https://raw.githubusercontent.com/bayanovbulat/AAPL/refs/heads/main/AAPLtime_series_Worldwide_20230101-0000_20260228-1453.csv')
https://raw.githubusercontent.com/bayanovbulat/AAPL/refs/heads/main/:
# Исходные данные Google Trends в период 230101_250101
AAPLtime_series_Worldwide_20230101-0000_20260228-1453.csv,
CSCOtime_series_Worldwide_20230101-0000_20260228-1500.csv,
MSFTtime_series_Worldwide_20230101-0000_20260228-1458.csv,
QCOMtime_series_Worldwide_20230101-0000_20260228-1459.csv
Задание:
1)	Выбрать три крупные компании, провести сравнительный анализ выбранных компаний в Google Trends в период с 01.01.2023-01.01.2025 по всему миру, затем скачать данные в csv файл;
2)	Загрузить csv файлы с данными о ценах акций выбранных компаний в период с 01.01.2023 по 01.01.2025 с периодичностью в неделю, затем с периодичностью в 1 день;
3)	С помощью инструмента pandas и sqlite3 загрузить с каждого csv файла данные в каждый файл базы данных database.
4)	Выполнить запрос SELECT из таблицы файла базы данных sqlite3, сортируя строки по дате.
5)	Выполнить запрос SELECT со средними, минимальными, максимальными значениями атрибутов <LOW> и <HIGH> за 2023 год и за 2024 год.
6)	Запросом SELECT определить несколько недель (дней) с наибольшим количеством торгов <VOL>.
7)	Запросом SELECT определить несколько недель (дней) с наибольшим значением максимальной цены сделки <HIGH>.
 
ЛАБОРАТОРНАЯ РАБОТА № 2
Графический вывод информации об исходных данных.
С помощью пакетов данных sqlite3, matplotlib, numpy, datetime отобразим исходные данные <LOW> и <HIGH> на одном графике.
Листинг программы №2.1 «graph.py»
import sqlite3
import matplotlib.pyplot as plt
import pandas as pd
# Загружаем данные через pandas для простоты
conn = sqlite3.connect('database.db')
df = pd.read_sql_query('''
    SELECT date, low_price, high_price 
    FROM weekly_prices 
    ORDER BY date
''', conn)
conn.close()
# Преобразуем дату
df['date'] = pd.to_datetime(df['date'], format='%y%m%d')
# Создаем график
plt.figure(figsize=(12, 6))
plt.plot(df['date'], df['low_price'], 'r-', label='LOW', linewidth=1)
plt.plot(df['date'], df['high_price'], 'g-', label='HIGH', linewidth=1)
plt.fill_between(df['date'], df['low_price'], df['high_price'], 
                 alpha=0.2, color='blue')
plt.title('Apple Inc. (AAPL) - Недельные цены LOW и HIGH', fontsize=14)
plt.xlabel('Дата')
plt.ylabel('Цена (USD)')
plt.legend()
plt.grid(True, alpha=0.3)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
Теперь продемонстрируем таблицу в 15 строк в формате html страницы.
Листинг программы №2.2 «html_report.py»
import sqlite3
from datetime import datetime
# Загружаем данные
conn = sqlite3.connect('database.db')
cursor = conn.cursor()
cursor.execute('''
    SELECT date, low_price, high_price 
    FROM weekly_prices 
    ORDER BY date DESC
    LIMIT 15
''')
data = cursor.fetchall()
conn.close()
# Создаем простую HTML таблицу
html_table_info = '''
<table style="width:70%; border-collapse:collapse; font-family: Arial;">
    <tr style="background-color:#C4C4C4; color:black;">
        <th style="padding:10px; border:1px solid #ddd;">Дата</th>
        <th style="padding:10px; border:1px solid #ddd;">LOW ($)</th>
        <th style="padding:10px; border:1px solid #ddd;">HIGH ($)</th>
        <th style="padding:10px; border:1px solid #ddd;">Размах ($)</th>
        <th style="padding:10px; border:1px solid #ddd;">%</th>
    </tr>
'''
for i, row in enumerate(data):
    date = str(row[0])
    low = row[1]
    high = row[2]
    spread = high - low
    percent = (spread / low * 100) if low > 0 else 0
    # Форматируем дату
    if len(date) == 6:
        formatted_date = f"20{date[0:2]}-{date[2:4]}-{date[4:6]}"
    else:
        formatted_date = date
    # Чередуем цвета строк
    bg_color = "#f2f2f2" if i % 2 == 0 else "white"
    html_table_info += f'''
    <tr style="background-color:{bg_color};">
        <td style="padding:8px; border:1px solid #ddd;">{formatted_date}</td>
        <td style="padding:8px; border:1px solid #ddd;">${low:.2f}</td>
        <td style="padding:8px; border:1px solid #ddd;">${high:.2f}</td>
        <td style="padding:8px; border:1px solid #ddd;">${spread:.2f}</td>
        <td style="padding:8px; border:1px solid #ddd;">{percent:.1f}%</td>
    </tr>
    '''
html_table_info += '</table>'
# Сохраняем
with open('report.html', 'w', encoding='utf-8') as f:
    f.write(html_table_info)
Помимо таблиц в html формате добавим рисунки в формате *.jpeg.
Листинг программы №2.3 «html_report.py»
# Создаем тег img с основными атрибутами
img_tag = '''
<img src="graph.jpeg"
     alt="График цен Apple Inc. (AAPL) - LOW и HIGH" 
     width="800"
     height="400"
     title="Apple Inc. - Недельные цены LOW и HIGH"
     style="border: 2px solid #4CAF50; border-radius: 8px; margin-left: 200px;">
'''
Задание:
1)	Отобразить на одном графике информацию <OPEN> от трех компаний;
2)	Отобразить на одном графике информацию <VOL> от трех компаний;
3)	Отобразить на одном графике информацию Google Trends от трех компаний, загрузив с csv файла;
4)	В хорошем стиле оформить html страницу с отчетом о таблицах и графиках, например, добавив теги <h1>, <p>, <i>, <b>, <hr> и т.д.
 
ЛАБОРАТОРНАЯ РАБОТА № 3
Статистический анализ временных рядов.
При анализе временных рядов предельно важным считается отслеживание линии тренда. Ниже визуально демонстрируется тренд объема торгов <VOL>.
Листинг программы №3.1 «trends.py»
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
# Загружаем данные
df = pd.read_csv('AAPL_230101_250101.csv', delimiter=';')
df['DATE'] = pd.to_datetime(df['<DATE>'], format='%y%m%d')
df['VOL_MILLIONS'] = df['<VOL>'] / 1_000_000
# Вычисляем тренд
x = range(len(df))
coeff = np.polyfit(x, df['VOL_MILLIONS'], 1)
trend = np.poly1d(coeff)
# Строим график
plt.figure(figsize=(12, 5))
plt.plot(df['DATE'], df['VOL_MILLIONS'], 'b-', alpha=0.7, linewidth=1, label='Объем торгов')
plt.plot(df['DATE'], trend(x), 'r-', linewidth=2.5, label='Линия тренда')
plt.title('Apple Inc. - Тренд объема торгов')
plt.xlabel('Дата')
plt.ylabel('Объем (млн акций)')
plt.legend()
plt.grid(True, alpha=0.3)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
Найдем корреляционную зависимость между значениями исходных данных (по атрибутам <OPEN> и <CLOSE>) и значениями, образованными инструментом Google Trends.
Листинг программы №3.2 «corr.py»
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
# 1. Загружаем данные
df_stock = pd.read_csv('AAPL_230101_250101.csv', delimiter=';')
df_google = pd.read_csv('time_series_Worldwide_20230101-0000_20260222-1041.csv')
# 2. Преобразуем даты
df_stock['DATE'] = pd.to_datetime(df_stock['<DATE>'], format='%y%m%d')
df_google['DATE'] = pd.to_datetime(df_google['Time'])
# 3. Берем только нужные колонки и агрегируем по месяцам
df_stock['YEAR_MONTH'] = df_stock['DATE'].dt.to_period('M')
df_google['YEAR_MONTH'] = df_google['DATE'].dt.to_period('M')
# 4. Группируем по месяцам
monthly_stock = df_stock.groupby('YEAR_MONTH')[['<OPEN>', '<CLOSE>']].mean().reset_index()
monthly_google = df_google.groupby('YEAR_MONTH')[['Apple']].mean().reset_index()
# 5. Объединяем данные
merged = pd.merge(monthly_stock, monthly_google, on='YEAR_MONTH')
# 6. Вычисляем корреляцию
corr_open = merged['<OPEN>'].corr(merged['Apple'])
corr_close = merged['<CLOSE>'].corr(merged['Apple'])
# 7. Выводим результаты
print(f"Корреляция OPEN vs Google Interest: {corr_open:.4f}")
print(f"Корреляция CLOSE vs Google Interest: {corr_close:.4f}")
# 8. Простой график
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
ax1.scatter(merged['Apple'], merged['<OPEN>'], alpha=0.6, color='blue')
ax1.set_xlabel('Google Interest (%)')
ax1.set_ylabel('OPEN Price ($)')
ax1.set_title(f'OPEN vs Google (r={corr_open:.3f})')
ax1.grid(True, alpha=0.3)
ax2.scatter(merged['Apple'], merged['<CLOSE>'], alpha=0.6, color='green')
ax2.set_xlabel('Google Interest (%)')
ax2.set_ylabel('CLOSE Price ($)')
ax2.set_title(f'CLOSE vs Google (r={corr_close:.3f})')
ax2.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
Отобразим гистограмму частот исходных значений по атрибутам <LOW> и <HIGH>.
Листинг программы №3.3 «statistics.py»
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
# Загружаем данные
df = pd.read_csv('AAPL_230101_250101.csv', delimiter=';')
# Извлекаем значения
low = df['<LOW>']
high = df['<HIGH>']
# Создаем гистограммы и получаем данные для построения линий
plt.figure(figsize=(12, 6))
# Для LOW
counts_low, bins_low = np.histogram(low, bins=20)
bin_centers_low = (bins_low[:-1] + bins_low[1:]) / 2
plt.plot(bin_centers_low, counts_low, 'b-o', linewidth=2.5, markersize=6, 
         label=f'LOW (n={len(low)}, μ=${low.mean():.2f})', markerfacecolor='white')
# Для HIGH
counts_high, bins_high = np.histogram(high, bins=20)
bin_centers_high = (bins_high[:-1] + bins_high[1:]) / 2
plt.plot(bin_centers_high, counts_high, 'r-s', linewidth=2.5, markersize=6, 
         label=f'HIGH (n={len(high)}, μ=${high.mean():.2f})', markerfacecolor='white')
# Настройки графика
plt.title('Полигоны частот: распределение цен LOW и HIGH', fontsize=14, fontweight='bold')
plt.xlabel('Цена ($)', fontsize=12)
plt.ylabel('Частота (количество недель)', fontsize=12)
plt.legend(loc='upper right')
plt.grid(True, alpha=0.3, linestyle='--')
# Добавляем вертикальные линии для средних
plt.axvline(low.mean(), color='blue', linestyle='--', alpha=0.7, linewidth=1)
plt.axvline(high.mean(), color='red', linestyle='--', alpha=0.7, linewidth=1)
plt.tight_layout()
plt.show()
Задание:
1)	Найти и построить функции линии тренда для исходных данных по атрибутам <LOW> и <HIGH> для трех компаний в указанный период времени;
2)	Найти и построить функции линии тренда для значений, образованных инструментом Google Trends, для трех компаний в указанный период времени;
3)	Вычислить корреляционную зависимость между значениями атрибута <VOL> и значениями исходных данных с Google Trends для трех компаний в указанный период;
4)	Отобразить гистограмму частот от исходных значений по всем атрибутам для трех компаний с указанием статистических показателей, например, математическое ожидание и стандартное отклонение;
5)	Выявить аномальные выбросы в выборке значений исходных данных;
6)	Ответить на вопрос: какие закономерности выявлены в результате статистического анализа?
7)	Создать отчет по полученным статистическим показателям и ответить на вопрос: к какой из компаний наблюдается нарастающий интерес со стороны пользователей?
 
ЛАБОРАТОРНАЯ РАБОТА № 4
Модели прогнозирования.
Ниже представлен программный код реализации нейросетевой модели для прогнозирования цен на акции компании AAPL на период с 01.01.2025 по 01.01.2026. Используемые инструменты: pandas, numpy, matplotlib, sklearn, tensorflow, keras. В обучающую выборку включены исходные данные за период с 01.01.2023 по 01.01.2025.
Листинг программы №4.1 «prediction.py»
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import MinMaxScaler
from sklearn.metrics import mean_absolute_error, mean_squared_error
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, LSTM
# Загружаем данные
df_train = pd.read_csv('AAPL_230101_250101.csv', delimiter=';')
df_test = pd.read_csv('AAPL_250101_260101.csv', delimiter=';')
# Нормализация
scaler = MinMaxScaler()
train_scaled = scaler.fit_transform(df_train[['<CLOSE>']])
test_scaled = scaler.transform(df_test[['<CLOSE>']])
# Создание последовательностей
def create_sequences(data, window=10):
    X, y = [], []
    for i in range(window, len(data)):
        X.append(data[i-window:i, 0])
        y.append(data[i, 0])
    return np.array(X), np.array(y
WINDOW = 8
X_train, y_train = create_sequences(train_scaled, WINDOW)
X_test, y_test = create_sequences(test_scaled, WINDOW)
X_train = X_train.reshape(X_train.shape[0], X_train.shape[1], 1)
X_test = X_test.reshape(X_test.shape[0], X_test.shape[1], 1)
# Создание модели
model = Sequential([
    LSTM(50, return_sequences=True, input_shape=(WINDOW, 1)),
    LSTM(50),
    Dense(1)
])
model.compile(optimizer='adam', loss='mse')
# Обучение
history = model.fit(X_train, y_train, epochs=50, validation_split=0.1, verbose=0)
# Прогнозы
train_pred = model.predict(X_train)
test_pred = model.predict(X_test)
# Обратное масштабирование
train_pred = scaler.inverse_transform(train_pred)
test_pred = scaler.inverse_transform(test_pred)
y_train_actual = scaler.inverse_transform(y_train.reshape(-1, 1))
y_test_actual = scaler.inverse_transform(y_test.reshape(-1, 1))
# Метрики
def print_metrics(y_true, y_pred, name):
    mae = mean_absolute_error(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mape = np.mean(np.abs((y_true - y_pred) / y_true)) * 100
    print(f"\n{name}: MAE=${mae:.2f}, RMSE=${rmse:.2f}, MAPE={mape:.2f}%")
print_metrics(y_train_actual, train_pred, "ОБУЧЕНИЕ")
print_metrics(y_test_actual, test_pred, "ТЕСТ")
# Визуализация
plt.figure(figsize=(14, 6))
plt.plot(df_test['<DATE>'].values[WINDOW:], y_test_actual, 'b-', label='Факт', linewidth=2)
plt.plot(df_test['<DATE>'].values[WINDOW:], test_pred, 'r--', label='Прогноз', linewidth=2)
plt.title('Прогнозирование цен Apple с помощью LSTM')
plt.xlabel('Дата')
plt.ylabel('Цена ($)')
plt.legend()
plt.grid(True, alpha=0.3)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
Продемонстрируем получившийся результат на рисунке, сгенерированном с помощью пакета данных matplotlib.
 
Рис. 4. Прогнозирование цен акций компании AAPL
Задание:
1)	Изучить вышеуказанный код реализации нейросетевой модели в задаче прогнозирования цен акций компании AAPL;
2)	Загрузить исходные данные на следующий год, которые будем считать эталонными для прогнозируемых значений (период с 01.01.2025 по 01.01.2026) для трех компаний;
3)	Построить несколько моделей прогнозирования значений исходных временных рядов по атрибутам <LOW> и <HIGH>, например: линейная регрессия (Linear Regression), ARIMA, случайный лес (Random Forest), XGBoost, SARIMA, нейросетевые модели;
4)	Дать оценку построенным моделям по показателям: MAE, RMSE, MAPE, R2;
5)	По вышеуказанным оценкам определить наилучшую модель прогнозирования;
6)	Учитывая данные Google Trends, ответить на вопрос: на ценные бумаги каких компаний стоит обратить внимание вкладчикам на будущий прогнозируемый год?
