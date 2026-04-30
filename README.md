# Audiobook Backend (Laravel)

This repository contains the backend API for the **Autism Audiobook** project.  
It is built using **Laravel**, **PHP**, and **MySQL**, and it provides API endpoints for the Flutter frontend to retrieve audiobook and content data.

---

## 📌 Project Status

🚧 Still under development  
Current backend focuses on:
- Audiobook data retrieval
- Content list API
- Content summary API
- MySQL database connection
- UUID-based audiobook identification

---

## 🛠 Tech Stack

- Laravel
- PHP
- MySQL
- phpMyAdmin
- XAMPP
- Composer

---

## 🚀 Backend Features

### Audiobook API
- Retrieve audiobook details by `audiobook_id`
- Return title, author, description, cover image, audio file, duration, category, and status

### Content Management API
- Retrieve all content list
- Retrieve content summary
- Support Flutter content management page
- Support Flutter story list page

### Database Integration
- Connect Laravel with MySQL database
- Store audiobook metadata
- Use UUID for `audiobook_id`

---

## 📁 Project Structure

```text
audiobook_backend/
│
├── app/
│   ├── Http/
│   │   └── Controllers/
│   │       └── Api/
│   │           ├── AudiobookController.php
│   │           └── ContentManagementController.php
│   │
│   └── Models/
│       └── Audiobook.php
│
├── bootstrap/
│   └── app.php
│
├── config/
│   ├── app.php
│   ├── database.php
│   └── filesystems.php
│
├── database/
│   ├── migrations/
│   ├── seeders/
│   └── factories/
│
├── public/
│   └── index.php
│
├── resources/
│   ├── css/
│   ├── js/
│   └── views/
│
├── routes/
│   ├── api.php
│   ├── web.php
│   └── console.php
│
├── storage/
│   ├── app/
│   ├── framework/
│   └── logs/
│       └── laravel.log
│
├── tests/
├── vendor/
│
├── .env
├── .env.example
├── artisan
├── composer.json
├── composer.lock
├── package.json
├── phpunit.xml
├── vite.config.js
└── README.md
```

---

## 📄 Important Files

### `routes/api.php`

Defines API routes used by the Flutter frontend.

Example:

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\AudiobookController;
use App\Http\Controllers\Api\ContentManagementController;

Route::get('/audiobooks/{audiobookId}', [AudiobookController::class, 'getAudiobookData']);

Route::prefix('content')->group(function () {
    Route::get('/list', [ContentManagementController::class, 'getContentList']);
    Route::get('/summary', [ContentManagementController::class, 'getContentSummary']);
});
```

Main API endpoints:

```text
GET /api/audiobooks/{audiobookId}
GET /api/content/list
GET /api/content/summary
```

---

### `app/Http/Controllers/Api/AudiobookController.php`

Handles audiobook-related API logic.

Responsibilities:
- Validate `audiobook_id`
- Retrieve audiobook from database
- Return JSON response to Flutter
- Log request information

Example:

```php
public function getAudiobookData(Request $request, string $audiobookId): JsonResponse
{
    $audiobook = Audiobook::where('audiobook_id', $audiobookId)->first();

    if (!$audiobook) {
        return response()->json([
            'status' => 'ERROR',
            'message' => 'Audiobook not found.'
        ], 404);
    }

    return response()->json([
        'status' => 'SUCCESS',
        'message' => 'Audiobook data retrieved successfully',
        'data' => [
            'audiobook_id' => $audiobook->audiobook_id,
            'title' => $audiobook->title,
            'author' => $audiobook->author,
            'audio_file' => $audiobook->audio_file,
            'cover_image' => $audiobook->cover_image,
        ]
    ]);
}
```

---

### `app/Http/Controllers/Api/ContentManagementController.php`

Handles content management API logic.

Responsibilities:
- Retrieve content list
- Retrieve content summary
- Return content data for Flutter story list and content management page

Example:

```php
public function getContentList(Request $request): JsonResponse
{
    $contents = Audiobook::orderBy('created_at', 'desc')->get();

    return response()->json([
        'status' => 'SUCCESS',
        'message' => 'Content list retrieved successfully',
        'data' => [
            'items' => $contents
        ]
    ]);
}
```

---

### `app/Models/Audiobook.php`

Laravel model for the `audiobooks` table.

Responsibilities:
- Represents audiobook database records
- Defines fillable fields
- Generates UUID for `audiobook_id`

Example:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Str;

class Audiobook extends Model
{
    use HasFactory;

    protected $table = 'audiobooks';

    protected $fillable = [
        'audiobook_id',
        'title',
        'author',
        'description',
        'topic',
        'category',
        'difficulty',
        'type',
        'content_text',
        'audio_file',
        'source_file',
        'cover_image',
        'duration_minutes',
        'language',
        'age_group',
        'tags',
        'is_generated',
        'is_user_uploaded',
        'status',
    ];

    protected $casts = [
        'is_generated' => 'boolean',
        'is_user_uploaded' => 'boolean',
        'duration_minutes' => 'integer',
    ];

    protected static function boot()
    {
        parent::boot();

        static::creating(function ($model) {
            if (!$model->audiobook_id) {
                $model->audiobook_id = (string) Str::uuid();
            }
        });
    }
}
```

---

### `.env`

Stores environment configuration.

Important settings:

```env
APP_NAME=AudiobookBackend
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=autism_audiobook
DB_USERNAME=root
DB_PASSWORD=
```

For Android emulator testing, `APP_URL` can be changed to:

```env
APP_URL=http://10.0.2.2:8000
```

After editing `.env`, run:

```bash
php artisan optimize:clear
```

---

### `storage/logs/laravel.log`

Laravel log file used for debugging.

Useful for checking:
- API request logs
- SQL errors
- Controller errors
- Route problems
- Server exceptions

Example log location:

```text
storage/logs/laravel.log
```

If API returns error, check this file first.

---

### `bootstrap/app.php`

In newer Laravel versions, `api.php` must be registered here.

Make sure API routing is enabled:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    api: __DIR__.'/../routes/api.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
)
```

If `api.php` is not registered, API routes will not appear in:

```bash
php artisan route:list
```

---

## ⚙️ Backend Setup

### 1. Install XAMPP

Download and install XAMPP.

Start:
- Apache
- MySQL

---

### 2. Install Composer

Download Composer from:

```text
https://getcomposer.org/download/
```

During installation, select PHP path:

```text
C:\xampp\php\php.exe
```

Check Composer:

```bash
composer -v
```

---

### 3. Clone Backend Repository

```bash
git clone https://github.com/scz912/audiobook_backend.git
cd audiobook_backend
```

If the project is inside XAMPP:

```bash
cd C:\xampp\htdocs\audiobook_backend
```

---

### 4. Install Laravel Dependencies

```bash
composer install
```

---

### 5. Create `.env`

Copy example file:

```bash
copy .env.example .env
```

Generate app key:

```bash
php artisan key:generate
```

---

### 6. Configure Database in `.env`

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=autism_audiobook
DB_USERNAME=root
DB_PASSWORD=
```

For default XAMPP:
- username: `root`
- password: empty

---

### 7. Clear Cache

```bash
php artisan optimize:clear
```

---

### 8. Run Laravel Server

```bash
php artisan serve
```

Default URL:

```text
http://127.0.0.1:8000
```

For Android emulator / same network testing:

```bash
php artisan serve --host=0.0.0.0 --port=8000
```

---

## 🗄 phpMyAdmin Database Setup

### 1. Open phpMyAdmin

```text
http://localhost/phpmyadmin
```

---

### 2. Create Database

Create a database named:

```text
autism_audiobook
```

---

### 3. Create `audiobooks` Table

```sql
CREATE TABLE audiobooks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    audiobook_id CHAR(36) NOT NULL UNIQUE,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NULL,
    description TEXT NULL,
    topic VARCHAR(100) NULL,
    category VARCHAR(100) NULL,
    difficulty ENUM('easy', 'medium', 'hard') DEFAULT 'easy',
    type ENUM('Audio', 'Text') DEFAULT 'Text',
    content_text LONGTEXT NULL,
    audio_file VARCHAR(1000) NULL,
    source_file VARCHAR(255) NULL,
    cover_image VARCHAR(255) NULL,
    duration_minutes INT NULL,
    language VARCHAR(50) DEFAULT 'English',
    age_group VARCHAR(50) NULL,
    tags TEXT NULL,
    is_generated TINYINT(1) DEFAULT 0,
    is_user_uploaded TINYINT(1) DEFAULT 1,
    status ENUM('available', 'processing', 'draft') DEFAULT 'available',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

### 4. Insert Sample Data

```sql
INSERT INTO audiobooks (
    audiobook_id,
    title,
    author,
    description,
    topic,
    category,
    difficulty,
    type,
    content_text,
    audio_file,
    cover_image,
    duration_minutes,
    language,
    age_group,
    tags,
    is_generated,
    is_user_uploaded,
    status
) VALUES (
    UUID(),
    'Ocean Friends',
    'Michael Chen',
    'A calming story about sea animals learning friendship.',
    'Friendship',
    'Animals',
    'easy',
    'Audio',
    'Once upon a time, under the blue ocean...',
    'storage/uploads/audio/ocean_friends.mp3',
    'storage/uploads/images/ocean_friends.jpg',
    12,
    'English',
    '7-9',
    'ocean, animals, friendship',
    0,
    1,
    'available'
);
```

---

## 🔗 API Testing

### Check Route List

```bash
php artisan route:list
```

Expected routes:

```text
GET api/audiobooks/{audiobookId}
GET api/content/list
GET api/content/summary
```

---

### Test API in Browser or Postman

Content list:

```text
GET http://127.0.0.1:8000/api/content/list
```

Content summary:

```text
GET http://127.0.0.1:8000/api/content/summary
```

Audiobook detail:

```text
GET http://127.0.0.1:8000/api/audiobooks/{audiobook_id}
```

Example:

```text
GET http://127.0.0.1:8000/api/audiobooks/2f818bb5-4178-11f1-b43a-e073e7f31f58
```

---

## 📦 Public Storage for Audio and Images

If using Laravel storage for audio or cover images, run:

```bash
php artisan storage:link
```

Recommended database paths:

```text
storage/uploads/audio/example.mp3
storage/uploads/images/example.jpg
```

Laravel should return public URLs so Flutter can load audio/images.

---

## ⚠️ Common Issues

### API route not found

Check:

```bash
php artisan route:list
```

If API routes do not appear, check `bootstrap/app.php` and make sure `api.php` is registered.

---

### Database connection error

Check `.env`:

```env
DB_DATABASE=autism_audiobook
DB_USERNAME=root
DB_PASSWORD=
```

Then run:

```bash
php artisan optimize:clear
```

---

### Class controller does not exist

Make sure controller namespace matches file location.

Correct file:

```text
app/Http/Controllers/Api/AudiobookController.php
```

Correct namespace:

```php
namespace App\Http\Controllers\Api;
```

---

### Audio cannot play in Flutter

Check:
- `audio_file` URL is public
- file exists
- `php artisan storage:link` has been run
- URL can open in browser
- for emulator, use correct host URL

---

### Check Laravel error log

```text
storage/logs/laravel.log
```

---

## 🚧 Current Limitations

- Authentication not fully implemented
- Upload content API still under development
- AI generation not implemented
- Audio files must be publicly accessible
- Error handling can still be improved

---

## 🔮 Future Improvements

- Add login and user authentication
- Add caregiver profile management
- Add child profile table
- Add upload audio/text API
- Add AI text-to-speech generation
- Add listening history
- Add recommendation system
- Add dashboard analytics

---

## 🎓 Project Purpose

This backend is developed as part of the **Autism Audiobook Final Year Project (FYP)**.

The backend supports the Flutter frontend by providing:
- Audiobook data
- Content list
- Content summary
- API-based communication
- MySQL database integration
