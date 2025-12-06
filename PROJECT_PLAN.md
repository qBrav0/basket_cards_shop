# 🏀 BasketCards - План розробки сайту

## Курсова робота: Інтернет-магазин колекційних баскетбольних карток

---

## 📋 1. Загальний опис проекту

**Назва:** BasketCards  
**Тип:** Інтернет-магазин колекційних баскетбольних карток  
**Технологія:** Django (Python)  
**Мета:** Створення повнофункціонального веб-додатку для продажу та управління колекційними баскетбольними картками

---

## 🛠 2. Технологічний стек

### Backend
- **Python 3.11+**
- **Django 5.0** - основний фреймворк
- **Django REST Framework** - для API (опціонально)
- **SQLite** (розробка) / **PostgreSQL** (продакшн)

### Frontend
- **HTML5 / CSS3**
- **Bootstrap 5** - основний CSS фреймворк (максимальне використання вбудованих класів)
- **JavaScript** - інтерактивність
- **HTMX** - динамічні оновлення без перезавантаження (опціонально)

> ⚠️ **Важливо:** Стилізація проекту базується на Bootstrap 5. Кастомний CSS використовується мінімально — лише для специфічних елементів, яких немає в Bootstrap.

### Додаткові бібліотеки
- **Pillow** - обробка зображень
- **django-crispy-forms** - стилізація форм
- **django-filter** - фільтрація товарів
- **django-allauth** - розширена автентифікація (опціонально)

---

## 📁 3. Структура проекту Django

```
basket_cards_shop/
│
├── manage.py
├── requirements.txt
├── README.md
│
├── config/                     # Головна конфігурація проекту
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
│
├── apps/
│   ├── accounts/               # Користувачі та автентифікація
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── forms.py
│   │   ├── urls.py
│   │   └── templates/
│   │
│   ├── cards/                  # Каталог карток
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── forms.py
│   │   ├── urls.py
│   │   ├── admin.py
│   │   └── templates/
│   │
│   ├── cart/                   # Кошик
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   └── templates/
│   │
│   ├── orders/                 # Замовлення
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── forms.py
│   │   ├── urls.py
│   │   └── templates/
│   │
│   └── core/                   # Загальні компоненти
│       ├── views.py
│       ├── urls.py
│       └── templates/
│
├── static/                     # Статичні файли
│   ├── css/
│   ├── js/
│   └── images/
│
├── media/                      # Завантажені файли (зображення карток)
│
└── templates/                  # Глобальні шаблони
    ├── base.html
    ├── navbar.html
    └── footer.html
```

---

## 🗄 4. Моделі даних (Database Schema)

### 4.1 Модель `User` (accounts)
```python
# Розширення стандартної моделі користувача
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    phone = models.CharField(max_length=20, blank=True)
    address = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
```

### 4.2 Модель `Category` (cards)
```python
class Category(models.Model):
    name = models.CharField(max_length=100)          # NBA, NCAA, Vintage, Rookie
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    image = models.ImageField(upload_to='categories/', blank=True)
```

### 4.3 Модель `Player` (cards)
```python
class Player(models.Model):
    name = models.CharField(max_length=200)          # LeBron James
    team = models.CharField(max_length=100)          # Los Angeles Lakers
    position = models.CharField(max_length=50)       # Small Forward
    jersey_number = models.IntegerField(null=True)
    bio = models.TextField(blank=True)
    photo = models.ImageField(upload_to='players/', blank=True)
```

### 4.4 Модель `Card` (cards) - Основна модель
```python
class Card(models.Model):
    CONDITION_CHOICES = [
        ('mint', 'Mint (Ідеальний)'),
        ('near_mint', 'Near Mint (Майже ідеальний)'),
        ('excellent', 'Excellent (Відмінний)'),
        ('good', 'Good (Хороший)'),
        ('fair', 'Fair (Задовільний)'),
    ]
    
    RARITY_CHOICES = [
        ('common', 'Common'),
        ('uncommon', 'Uncommon'),
        ('rare', 'Rare'),
        ('legendary', 'Legendary'),
        ('one_of_one', '1/1'),
    ]
    
    title = models.CharField(max_length=300)
    slug = models.SlugField(unique=True)
    player = models.ForeignKey(Player, on_delete=models.CASCADE)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    
    year = models.IntegerField()                     # Рік випуску
    brand = models.CharField(max_length=100)         # Panini, Topps, Upper Deck
    series = models.CharField(max_length=200)        # Prizm, Mosaic, Hoops
    card_number = models.CharField(max_length=50)    # Номер картки в серії
    
    condition = models.CharField(max_length=20, choices=CONDITION_CHOICES)
    rarity = models.CharField(max_length=20, choices=RARITY_CHOICES)
    is_autographed = models.BooleanField(default=False)
    is_graded = models.BooleanField(default=False)
    grade = models.CharField(max_length=20, blank=True)  # PSA 10, BGS 9.5
    
    price = models.DecimalField(max_digits=10, decimal_places=2)
    discount_price = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
    stock = models.PositiveIntegerField(default=1)
    
    image = models.ImageField(upload_to='cards/')
    image_back = models.ImageField(upload_to='cards/', blank=True)
    
    description = models.TextField()
    is_featured = models.BooleanField(default=False)
    is_active = models.BooleanField(default=True)
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    views = models.PositiveIntegerField(default=0)
```

### 4.5 Модель `CartItem` (cart)
```python
class Cart(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, null=True, blank=True)
    session_key = models.CharField(max_length=40, null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

class CartItem(models.Model):
    cart = models.ForeignKey(Cart, on_delete=models.CASCADE)
    card = models.ForeignKey(Card, on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField(default=1)
```

### 4.6 Модель `Order` (orders)
```python
class Order(models.Model):
    STATUS_CHOICES = [
        ('pending', 'Очікує підтвердження'),
        ('confirmed', 'Підтверджено'),
        ('shipped', 'Відправлено'),
        ('delivered', 'Доставлено'),
        ('cancelled', 'Скасовано'),
    ]
    
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    order_number = models.CharField(max_length=50, unique=True)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='pending')
    
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    email = models.EmailField()
    phone = models.CharField(max_length=20)
    address = models.TextField()
    city = models.CharField(max_length=100)
    postal_code = models.CharField(max_length=20)
    
    total_amount = models.DecimalField(max_digits=10, decimal_places=2)
    shipping_cost = models.DecimalField(max_digits=10, decimal_places=2, default=0)
    
    notes = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE, related_name='items')
    card = models.ForeignKey(Card, on_delete=models.SET_NULL, null=True)
    quantity = models.PositiveIntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
```

### 4.7 Модель `Review` (cards)
```python
class Review(models.Model):
    card = models.ForeignKey(Card, on_delete=models.CASCADE, related_name='reviews')
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    rating = models.IntegerField(choices=[(i, i) for i in range(1, 6)])
    comment = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
```

### 4.8 Модель `Wishlist` (cards)
```python
class Wishlist(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    card = models.ForeignKey(Card, on_delete=models.CASCADE)
    added_at = models.DateTimeField(auto_now_add=True)
```

---

## ⚙️ 5. Функціональні вимоги

### 5.1 Для відвідувачів (неавторизованих)
- [x] Перегляд головної сторінки з featured картками
- [x] Перегляд каталогу карток
- [x] Фільтрація за категорією, гравцем, ціною, станом, рідкістю
- [x] Пошук карток
- [x] **Пагінація на сторінках каталогу** (12-24 картки на сторінку)
- [x] Перегляд детальної сторінки картки
- [x] Додавання товарів у кошик (session-based)
- [x] Реєстрація та авторизація

### 5.2 Для авторизованих користувачів
- [x] Всі функції відвідувачів
- [x] Особистий кабінет (профіль)
- [x] Редагування профілю
- [x] Оформлення замовлення
- [x] Історія замовлень
- [x] Список бажань (Wishlist)
- [x] Залишати відгуки на картки
- [x] Відстеження статусу замовлення

### 5.3 Для адміністратора (Django Admin)
- [x] CRUD операції для всіх моделей
- [x] Управління замовленнями
- [x] Управління користувачами
- [x] Статистика продажів
- [x] Модерація відгуків

---

## 📄 6. Сторінки сайту

| Сторінка | URL | Опис |
|----------|-----|------|
| Головна | `/` | Банер, featured картки, категорії |
| Каталог | `/cards/` | Список карток з фільтрами |
| Картка товару | `/cards/<slug>/` | Детальна інформація |
| Категорії | `/categories/` | Список категорій |
| Категорія | `/categories/<slug>/` | Картки категорії |
| Гравці | `/players/` | Список гравців |
| Гравець | `/players/<slug>/` | Картки гравця |
| Пошук | `/search/` | Результати пошуку |
| Кошик | `/cart/` | Вміст кошика |
| Оформлення | `/checkout/` | Форма замовлення |
| Реєстрація | `/accounts/register/` | Форма реєстрації |
| Вхід | `/accounts/login/` | Форма входу |
| Профіль | `/accounts/profile/` | Особистий кабінет |
| Замовлення | `/orders/` | Історія замовлень |
| Замовлення | `/orders/<id>/` | Деталі замовлення |
| Wishlist | `/wishlist/` | Список бажань |
| Про нас | `/about/` | Інформація про магазин |
| Контакти | `/contact/` | Контактна форма |

---

## 📅 7. План розробки (Етапи)

### Етап 1: Налаштування проекту (1-2 дні)
- [ ] Створення віртуального середовища
- [ ] Встановлення Django та залежностей
- [ ] Налаштування структури проекту
- [ ] Налаштування settings.py
- [ ] Створення базових шаблонів (base.html, navbar, footer)
- [ ] Підключення Bootstrap 5
- [ ] Налаштування статичних файлів та медіа

### Етап 2: Модуль автентифікації (2-3 дні)
- [ ] Створення app `accounts`
- [ ] Модель UserProfile
- [ ] Реєстрація користувачів
- [ ] Авторизація (login/logout)
- [ ] Відновлення паролю
- [ ] Сторінка профілю
- [ ] Редагування профілю

### Етап 3: Каталог карток (3-4 дні)
- [ ] Створення app `cards`
- [ ] Моделі: Category, Player, Card
- [ ] Налаштування Django Admin
- [ ] Список карток (ListView)
- [ ] Детальна сторінка картки (DetailView)
- [ ] Фільтрація (django-filter)
- [ ] Пошук
- [ ] **Пагінація** (Django Paginator, 12-24 карток на сторінку)
- [ ] Сторінки категорій та гравців

### Етап 4: Кошик (2-3 дні)
- [ ] Створення app `cart`
- [ ] Моделі Cart, CartItem
- [ ] Додавання в кошик (AJAX)
- [ ] Оновлення кількості
- [ ] Видалення з кошика
- [ ] Підрахунок суми
- [ ] Session-based кошик для гостей

### Етап 5: Замовлення (3-4 дні)
- [ ] Створення app `orders`
- [ ] Моделі Order, OrderItem
- [ ] Форма оформлення замовлення
- [ ] Валідація даних
- [ ] Створення замовлення
- [ ] Підтвердження замовлення (email)
- [ ] Історія замовлень
- [ ] Деталі замовлення

### Етап 6: Додатковий функціонал (2-3 дні)
- [ ] Система відгуків (Review)
- [ ] Список бажань (Wishlist)
- [ ] Сортування карток
- [ ] Featured картки на головній
- [ ] Рекомендації "Схожі картки"
- [ ] Лічильник переглядів

### Етап 7: UI/UX та дизайн (2-3 дні)
- [ ] Оформлення головної сторінки (Bootstrap Hero, Carousel)
- [ ] Стилізація каталогу (Bootstrap Grid, Cards)
- [ ] Картки товарів (Bootstrap Card component)
- [ ] Адаптивний дизайн (Bootstrap breakpoints: sm, md, lg, xl)
- [ ] Bootstrap Offcanvas для фільтрів на мобільних
- [ ] Bootstrap Icons для іконок
- [ ] Мінімальний кастомний CSS (лише кольори та шрифти)

### Етап 8: Тестування та деплой (1-2 дні)
- [ ] Написання тестів (pytest-django)
- [ ] Перевірка всього функціоналу
- [ ] Оптимізація запитів до БД
- [ ] Підготовка до деплою
- [ ] Документація

---

## 📑 8. Пагінація (детальний опис)

### Сторінки з пагінацією:
- `/cards/` — каталог усіх карток
- `/categories/<slug>/` — картки категорії
- `/players/<slug>/` — картки гравця
- `/search/` — результати пошуку
- `/orders/` — історія замовлень

### Реалізація в Django (Class-Based Views):
```python
# views.py
from django.views.generic import ListView

class CardListView(ListView):
    model = Card
    template_name = 'cards/card_list.html'
    context_object_name = 'cards'
    paginate_by = 12  # Кількість карток на сторінку
    ordering = ['-created_at']
```

### Реалізація в Django (Function-Based Views):
```python
# views.py
from django.core.paginator import Paginator

def card_list(request):
    cards = Card.objects.filter(is_active=True).order_by('-created_at')
    paginator = Paginator(cards, 12)  # 12 карток на сторінку
    
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    return render(request, 'cards/card_list.html', {'page_obj': page_obj})
```

### Шаблон пагінації (Bootstrap 5):
```html
<!-- templates/includes/pagination.html -->
{% if page_obj.has_other_pages %}
<nav aria-label="Навігація по сторінках">
    <ul class="pagination justify-content-center">
        {% if page_obj.has_previous %}
            <li class="page-item">
                <a class="page-link" href="?page=1">&laquo; Перша</a>
            </li>
            <li class="page-item">
                <a class="page-link" href="?page={{ page_obj.previous_page_number }}">Попередня</a>
            </li>
        {% endif %}

        {% for num in page_obj.paginator.page_range %}
            {% if page_obj.number == num %}
                <li class="page-item active">
                    <span class="page-link">{{ num }}</span>
                </li>
            {% elif num > page_obj.number|add:'-3' and num < page_obj.number|add:'3' %}
                <li class="page-item">
                    <a class="page-link" href="?page={{ num }}">{{ num }}</a>
                </li>
            {% endif %}
        {% endfor %}

        {% if page_obj.has_next %}
            <li class="page-item">
                <a class="page-link" href="?page={{ page_obj.next_page_number }}">Наступна</a>
            </li>
            <li class="page-item">
                <a class="page-link" href="?page={{ page_obj.paginator.num_pages }}">Остання &raquo;</a>
            </li>
        {% endif %}
    </ul>
</nav>
<p class="text-center text-muted">
    Сторінка {{ page_obj.number }} з {{ page_obj.paginator.num_pages }} 
    (всього {{ page_obj.paginator.count }} карток)
</p>
{% endif %}
```

### Пагінація з фільтрами (збереження GET-параметрів):
```html
<!-- Для збереження фільтрів при переході між сторінками -->
<a href="?page={{ num }}&{{ request.GET.urlencode }}">{{ num }}</a>
```

---

## 🎨 9. Дизайн-концепція (Bootstrap 5)

### Підключення Bootstrap 5
```html
<!-- В base.html -->
<head>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Bootstrap Icons -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.1/font/bootstrap-icons.css" rel="stylesheet">
    <!-- Google Fonts (опціонально) -->
    <link href="https://fonts.googleapis.com/css2?family=Oswald:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Мінімальний кастомний CSS -->
    <link href="{% static 'css/custom.css' %}" rel="stylesheet">
</head>
<body>
    ...
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
```

### Кастомізація кольорів Bootstrap (custom.css)
```css
/* Мінімальний кастомний CSS - лише перевизначення змінних Bootstrap */
:root {
    --bs-primary: #E65100;       /* Помаранчевий (баскетбольний м'яч) */
    --bs-secondary: #1A237E;     /* Темно-синій */
    --bs-warning: #FFD700;       /* Золотий */
}

/* Кастомний шрифт для заголовків */
h1, h2, h3, .display-1, .display-2, .display-3 {
    font-family: 'Oswald', sans-serif;
}
```

### Основні Bootstrap-класи для проекту

#### Навігація (Navbar)
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark sticky-top">
    <div class="container">
        <a class="navbar-brand fw-bold" href="/">🏀 BasketCards</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav me-auto">
                <li class="nav-item"><a class="nav-link" href="/cards/">Каталог</a></li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" data-bs-toggle="dropdown">Категорії</a>
                    <ul class="dropdown-menu">...</ul>
                </li>
            </ul>
            <a href="/cart/" class="btn btn-outline-warning position-relative">
                <i class="bi bi-cart3"></i>
                <span class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger">3</span>
            </a>
        </div>
    </div>
</nav>
```

#### Картка товару (Card Component)
```html
<div class="card h-100 shadow-sm">
    <span class="badge bg-danger position-absolute top-0 end-0 m-2">-20%</span>
    <img src="{{ card.image.url }}" class="card-img-top" alt="{{ card.title }}">
    <div class="card-body d-flex flex-column">
        <h5 class="card-title text-truncate">{{ card.title }}</h5>
        <p class="card-text text-muted small">{{ card.player.name }}</p>
        <div class="mt-auto">
            <p class="card-text">
                <span class="text-decoration-line-through text-muted">{{ card.price }} грн</span>
                <span class="fs-5 fw-bold text-danger">{{ card.discount_price }} грн</span>
            </p>
            <a href="/cards/{{ card.slug }}/" class="btn btn-primary w-100">
                <i class="bi bi-eye"></i> Детальніше
            </a>
        </div>
    </div>
</div>
```

#### Сітка каталогу (Grid System)
```html
<div class="container py-4">
    <div class="row row-cols-1 row-cols-sm-2 row-cols-md-3 row-cols-lg-4 g-4">
        {% for card in cards %}
        <div class="col">
            {% include 'cards/card_item.html' %}
        </div>
        {% endfor %}
    </div>
</div>
```

#### Фільтри (Offcanvas на мобільних)
```html
<!-- Кнопка фільтрів на мобільних -->
<button class="btn btn-outline-secondary d-lg-none mb-3" type="button" data-bs-toggle="offcanvas" data-bs-target="#filters">
    <i class="bi bi-funnel"></i> Фільтри
</button>

<!-- Бічна панель фільтрів -->
<div class="offcanvas-lg offcanvas-start" id="filters">
    <div class="offcanvas-header">
        <h5 class="offcanvas-title">Фільтри</h5>
        <button type="button" class="btn-close" data-bs-dismiss="offcanvas"></button>
    </div>
    <div class="offcanvas-body">
        <form method="GET">
            <div class="mb-3">
                <label class="form-label fw-bold">Категорія</label>
                <select class="form-select" name="category">...</select>
            </div>
            <div class="mb-3">
                <label class="form-label fw-bold">Ціна</label>
                <input type="number" class="form-control" name="min_price" placeholder="Від">
                <input type="number" class="form-control mt-2" name="max_price" placeholder="До">
            </div>
            <button type="submit" class="btn btn-primary w-100">Застосувати</button>
        </form>
    </div>
</div>
```

#### Badges для рідкості та стану
```html
<!-- Рідкість -->
<span class="badge bg-secondary">Common</span>
<span class="badge bg-info">Uncommon</span>
<span class="badge bg-primary">Rare</span>
<span class="badge bg-warning text-dark">Legendary</span>
<span class="badge bg-danger">1/1</span>

<!-- Стан -->
<span class="badge rounded-pill bg-success">Mint</span>
<span class="badge rounded-pill bg-primary">Near Mint</span>
```

#### Кнопки (Buttons)
```html
<button class="btn btn-primary">Додати в кошик</button>
<button class="btn btn-outline-danger"><i class="bi bi-heart"></i></button>
<button class="btn btn-success btn-lg w-100">Оформити замовлення</button>
```

#### Форми (Forms з crispy-forms)
```html
<!-- Використання crispy-forms з Bootstrap 5 -->
{% load crispy_forms_tags %}
<form method="POST" class="needs-validation" novalidate>
    {% csrf_token %}
    {{ form|crispy }}
    <button type="submit" class="btn btn-primary">Зберегти</button>
</form>
```

#### Сповіщення (Alerts/Toasts)
```html
<!-- Django Messages з Bootstrap -->
{% for message in messages %}
<div class="alert alert-{{ message.tags }} alert-dismissible fade show" role="alert">
    {{ message }}
    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
</div>
{% endfor %}
```

### Корисні Bootstrap утиліти
| Клас | Опис |
|------|------|
| `container`, `container-fluid` | Контейнери |
| `row`, `col-*` | Сітка |
| `d-flex`, `justify-content-*`, `align-items-*` | Flexbox |
| `text-center`, `text-muted`, `fw-bold` | Текст |
| `py-*`, `px-*`, `my-*`, `mx-*` | Відступи |
| `shadow`, `shadow-sm`, `shadow-lg` | Тіні |
| `rounded`, `rounded-pill` | Заокруглення |
| `d-none`, `d-md-block` | Адаптивне приховування |
| `position-relative`, `position-absolute` | Позиціонування |

### Bootstrap Icons (часто використовувані)
```html
<i class="bi bi-cart3"></i>           <!-- Кошик -->
<i class="bi bi-heart"></i>           <!-- Вішліст -->
<i class="bi bi-heart-fill"></i>      <!-- Вішліст (заповнений) -->
<i class="bi bi-search"></i>          <!-- Пошук -->
<i class="bi bi-person"></i>          <!-- Профіль -->
<i class="bi bi-star-fill"></i>       <!-- Рейтинг -->
<i class="bi bi-funnel"></i>          <!-- Фільтри -->
<i class="bi bi-sort-down"></i>       <!-- Сортування -->
<i class="bi bi-box-seam"></i>        <!-- Замовлення -->
<i class="bi bi-truck"></i>           <!-- Доставка -->
```

---

## 📦 10. Файл requirements.txt

```txt
Django>=5.0
Pillow>=10.0
django-crispy-forms>=2.1
crispy-bootstrap5>=2024.2
django-filter>=23.5
python-dotenv>=1.0
```

### Налаштування crispy-forms для Bootstrap 5 (settings.py)
```python
INSTALLED_APPS = [
    ...
    'crispy_forms',
    'crispy_bootstrap5',
    ...
]

# Crispy Forms
CRISPY_ALLOWED_TEMPLATE_PACKS = "bootstrap5"
CRISPY_TEMPLATE_PACK = "bootstrap5"
```

---

## 🔒 11. Безпека

- [ ] CSRF захист (вбудований в Django)
- [ ] XSS захист
- [ ] SQL Injection захист (Django ORM)
- [ ] Валідація форм
- [ ] Хешування паролів
- [ ] HTTPS (для продакшн)
- [ ] Захист від brute-force (django-axes опціонально)

---


## ✅ Чек-лист готовності проекту

- [ ] Проект запускається без помилок
- [ ] Всі сторінки відображаються коректно
- [ ] Реєстрація/авторизація працює
- [ ] Каталог з фільтрацією працює
- [ ] Кошик функціонує
- [ ] Замовлення оформлюються
- [ ] Admin-панель налаштована
- [ ] Адаптивний дизайн
- [ ] Немає критичних помилок в консолі
- [ ] Код документований

---

**Автор:** [Ваше ім'я]  
**Дата створення:** Грудень 2024  
**Версія:** 1.0

