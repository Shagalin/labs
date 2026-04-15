
# Лабораторная работа №3: Детекция движения и траектория мыши

## Цель работы
Выделить движущийся объект (мышь) из видеозаписи с помощью метода вычитания фона, построить его бинарную маску, найти центр масс на каждом кадре и построить траекторию движения.

## Используемые методы
- **Background subtraction** (вычитание фона): средний фон по кадрам и динамический фон.
- **Морфологические операции**: opening (удаление шума), closing (заполнение дыр).
- **Моменты изображения**: вычисление центра масс объекта.
- **Медианный фильтр**: сглаживание кадров перед обработкой.

## Входные данные
- Видео: `mouse_1.avi` (~4000+ кадров, мышь бегает по белому фону).

## Основной алгоритм

### 1. Загрузка и чтение видео
```python
cap = cv2.VideoCapture("mouse_1.avi")
```
Читаем все кадры в список `frames`, переводим в grayscale.

### 2. Построение модели фона
**Средний фон** (`sr_mod_fon`):
```python
sum_arr[i,j] += image[i,j]  # сумма по пикселям всех кадров
sum_arr /= num_frames       # среднее значение
```
Для каждого пикселя берётся среднее по времени.

**Динамический фон** (`mod_fon`):
```python
mod_fon = 0.5 * current + 0.5 * old_fon  # экспоненциальное сглаживание
```

### 3. Вычитание фона
```python
diff_image = current_frame - background  # cv2.addWeighted(..., 1, -1, 0)
```
Движение становится ярким, фон ≈ 0.

### 4. Сегментация объекта
```python
thresh = cv2.threshold(diff_image, 20, 255, cv2.THRESH_BINARY)[1]
```
Бинаризация: >20 → белый (объект), ≤20 → чёрный (фон).

### 5. Морфологическая очистка
```python
thresh = cv2.morphologyEx(thresh, cv2.MORPH_OPEN, kernel)  # убирает шум
thresh = cv2.morphologyEx(thresh, cv2.MORPH_CLOSE, kernel) # заполняет дырки
```

### 6. Центр масс
```python
mu = cv2.moments(thresh)
mc = (mu['m10']/mu['m00'], mu['m01']/mu['m00'])  # центр масс бинарной фигуры
```
Фильтрация: игнорируем нулевые центры и резкие скачки (>30 пикселей).

### 7. Траектория
```python
plt.plot(D1, D2, marker='o')  # X и Y координаты центров масс по кадрам
```

## Дополнительные функции

### `median_image(frames)`
Строит фон через пер-пиксельные гистограммы:
```python
for пиксель (j,k):
    hist[frames[i][j][k]] += 1  # гистограмма яркостей по кадрам
    newimg[j,k] = np.argmax(hist)  # наиболее частое значение (мода)
```

### Медианный фильтр
```python
cv2.medianBlur(img, 7)  # медиана в окне 7×7, убирает шум
```

## Результаты
- Бинарная маска мыши на каждом кадре.
- Красная точка — центр масс.
- График траектории движения мыши.
- Видео с наложенной траекторией (`myvideo.avi`).

<img width="814" height="678" alt="изображение" src="https://github.com/user-attachments/assets/32c032d1-436b-4690-9245-59a175155324" />

<img width="587" height="221" alt="изображение" src="https://github.com/user-attachments/assets/318e64e9-7279-48bd-8a93-521467cf8d14" />

<img width="611" height="461" alt="изображение" src="https://github.com/user-attachments/assets/d7fb78a4-1758-4d6e-9b42-365dac2b48b1" />

Фон


<img width="578" height="470" alt="изображение" src="https://github.com/user-attachments/assets/b0df97ee-5f44-46d2-853c-ab18d20e902a" />

Сравнение

<img width="739" height="302" alt="изображение" src="https://github.com/user-attachments/assets/759d9401-b18b-4c98-b01d-61e664a3a416" />

Результат вычитания

<img width="555" height="421" alt="изображение" src="https://github.com/user-attachments/assets/1e5c1341-13b1-463e-90ae-16bac4c3cba7" />

Результат работы после морфологической операции закрытия (эрозия + делатация) + бинаризация

<img width="537" height="408" alt="изображение" src="https://github.com/user-attachments/assets/04b9b898-0a8d-4dfd-b4d5-c127c4a3a329" />

Перезапусти и вставь картинку с центром масс
<img width="590" height="487" alt="изображение" src="https://github.com/user-attachments/assets/36005745-75f3-43a4-aa60-663604e99fb4" />

И финальная траектория
<img width="589" height="443" alt="изображение" src="https://github.com/user-attachments/assets/2ac0db06-9bf7-4f56-8ebf-581d809a7667" />
