# Laravel 

Attorney management system using Laravel, PHP, MySQL, Blade, and resource controllers.

## 1. Database Design

### Main Entities

* `clients`: people or companies receiving legal services
* `attorneys`: lawyers working on cases
* `practice_areas`: criminal law, family law, corporate law, immigration, etc.
* `legal_cases`: case files managed by the firm
* `case_attorney`: pivot table for assigning attorneys to cases
* `hearings`: court hearings linked to cases
* `appointments`: client/attorney meetings
* `documents`: uploaded or referenced legal documents
* `invoices`: billing records for cases
* `payments`: payment records against invoices

---

# 2. Artisan Commands

Run these commands:

```bash
php artisan make:model Client -m
php artisan make:model Attorney -m
php artisan make:model PracticeArea -m
php artisan make:model LegalCase -m
php artisan make:model Hearing -m
php artisan make:model Appointment -m
php artisan make:model Document -m
php artisan make:model Invoice -m
php artisan make:model Payment -m

php artisan make:migration create_case_attorney_table

php artisan make:controller ClientController --resource
php artisan make:controller AttorneyController --resource
php artisan make:controller PracticeAreaController --resource
php artisan make:controller LegalCaseController --resource
php artisan make:controller HearingController --resource
php artisan make:controller AppointmentController --resource
php artisan make:controller DocumentController --resource
php artisan make:controller InvoiceController --resource
php artisan make:controller PaymentController --resource
```

---

# 3. Migrations

## database/migrations/create_clients_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('clients', function (Blueprint $table) {
            $table->id();
            $table->string('client_type')->default('individual'); // individual, company
            $table->string('first_name')->nullable();
            $table->string('last_name')->nullable();
            $table->string('company_name')->nullable();
            $table->string('email')->unique();
            $table->string('phone')->nullable();
            $table->string('address')->nullable();
            $table->string('city')->nullable();
            $table->string('state')->nullable();
            $table->string('postal_code')->nullable();
            $table->string('country')->nullable();
            $table->text('notes')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('clients');
    }
};
```

## database/migrations/create_attorneys_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('attorneys', function (Blueprint $table) {
            $table->id();
            $table->string('first_name');
            $table->string('last_name');
            $table->string('email')->unique();
            $table->string('phone')->nullable();
            $table->string('bar_number')->unique()->nullable();
            $table->string('position')->nullable();
            $table->decimal('hourly_rate', 10, 2)->default(0);
            $table->boolean('is_active')->default(true);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('attorneys');
    }
};
```

## database/migrations/create_practice_areas_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('practice_areas', function (Blueprint $table) {
            $table->id();
            $table->string('name')->unique();
            $table->text('description')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('practice_areas');
    }
};
```

## database/migrations/create_legal_cases_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('legal_cases', function (Blueprint $table) {
            $table->id();
            $table->foreignId('client_id')->constrained()->cascadeOnDelete();
            $table->foreignId('practice_area_id')->nullable()->constrained()->nullOnDelete();
            $table->string('case_number')->unique();
            $table->string('title');
            $table->text('description')->nullable();
            $table->string('court_name')->nullable();
            $table->string('opposing_party')->nullable();
            $table->string('opposing_counsel')->nullable();
            $table->enum('status', ['open', 'pending', 'closed', 'archived'])->default('open');
            $table->enum('priority', ['low', 'medium', 'high', 'urgent'])->default('medium');
            $table->date('opened_at')->nullable();
            $table->date('closed_at')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('legal_cases');
    }
};
```

## database/migrations/create_case_attorney_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('case_attorney', function (Blueprint $table) {
            $table->id();
            $table->foreignId('legal_case_id')->constrained('legal_cases')->cascadeOnDelete();
            $table->foreignId('attorney_id')->constrained()->cascadeOnDelete();
            $table->string('role')->default('assigned'); // lead, associate, paralegal, assigned
            $table->timestamps();

            $table->unique(['legal_case_id', 'attorney_id']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('case_attorney');
    }
};
```

## database/migrations/create_hearings_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('hearings', function (Blueprint $table) {
            $table->id();
            $table->foreignId('legal_case_id')->constrained('legal_cases')->cascadeOnDelete();
            $table->string('title');
            $table->string('courtroom')->nullable();
            $table->dateTime('scheduled_at');
            $table->enum('status', ['scheduled', 'completed', 'adjourned', 'cancelled'])->default('scheduled');
            $table->text('notes')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('hearings');
    }
};
```

## database/migrations/create_appointments_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('appointments', function (Blueprint $table) {
            $table->id();
            $table->foreignId('client_id')->constrained()->cascadeOnDelete();
            $table->foreignId('attorney_id')->nullable()->constrained()->nullOnDelete();
            $table->foreignId('legal_case_id')->nullable()->constrained('legal_cases')->nullOnDelete();
            $table->string('title');
            $table->dateTime('starts_at');
            $table->dateTime('ends_at')->nullable();
            $table->enum('status', ['scheduled', 'completed', 'cancelled', 'no_show'])->default('scheduled');
            $table->string('location')->nullable();
            $table->text('notes')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('appointments');
    }
};
```

## database/migrations/create_documents_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('documents', function (Blueprint $table) {
            $table->id();
            $table->foreignId('legal_case_id')->constrained('legal_cases')->cascadeOnDelete();
            $table->string('title');
            $table->string('document_type')->nullable();
            $table->string('file_path')->nullable();
            $table->text('description')->nullable();
            $table->date('received_at')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('documents');
    }
};
```

## database/migrations/create_invoices_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('invoices', function (Blueprint $table) {
            $table->id();
            $table->foreignId('client_id')->constrained()->cascadeOnDelete();
            $table->foreignId('legal_case_id')->nullable()->constrained('legal_cases')->nullOnDelete();
            $table->string('invoice_number')->unique();
            $table->decimal('subtotal', 10, 2)->default(0);
            $table->decimal('tax', 10, 2)->default(0);
            $table->decimal('total', 10, 2)->default(0);
            $table->enum('status', ['draft', 'sent', 'paid', 'overdue', 'cancelled'])->default('draft');
            $table->date('issued_at')->nullable();
            $table->date('due_at')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('invoices');
    }
};
```

## database/migrations/create_payments_table.php

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('payments', function (Blueprint $table) {
            $table->id();
            $table->foreignId('invoice_id')->constrained()->cascadeOnDelete();
            $table->decimal('amount', 10, 2);
            $table->string('payment_method')->nullable();
            $table->string('reference_number')->nullable();
            $table->date('paid_at');
            $table->text('notes')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('payments');
    }
};
```

Run migrations:

```bash
php artisan migrate
```

---

# 4. Models

## app/Models/Client.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Client extends Model
{
    use HasFactory;

    protected $fillable = [
        'client_type', 'first_name', 'last_name', 'company_name', 'email', 'phone',
        'address', 'city', 'state', 'postal_code', 'country', 'notes'
    ];

    public function legalCases()
    {
        return $this->hasMany(LegalCase::class);
    }

    public function appointments()
    {
        return $this->hasMany(Appointment::class);
    }

    public function invoices()
    {
        return $this->hasMany(Invoice::class);
    }

    public function getDisplayNameAttribute(): string
    {
        return $this->client_type === 'company'
            ? (string) $this->company_name
            : trim($this->first_name . ' ' . $this->last_name);
    }
}
```

## app/Models/Attorney.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Attorney extends Model
{
    use HasFactory;

    protected $fillable = [
        'first_name', 'last_name', 'email', 'phone', 'bar_number',
        'position', 'hourly_rate', 'is_active'
    ];

    protected $casts = [
        'is_active' => 'boolean',
        'hourly_rate' => 'decimal:2',
    ];

    public function legalCases()
    {
        return $this->belongsToMany(LegalCase::class, 'case_attorney')
            ->withPivot('role')
            ->withTimestamps();
    }

    public function appointments()
    {
        return $this->hasMany(Appointment::class);
    }

    public function getFullNameAttribute(): string
    {
        return trim($this->first_name . ' ' . $this->last_name);
    }
}
```

## app/Models/PracticeArea.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class PracticeArea extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'description'];

    public function legalCases()
    {
        return $this->hasMany(LegalCase::class);
    }
}
```

## app/Models/LegalCase.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class LegalCase extends Model
{
    use HasFactory;

    protected $fillable = [
        'client_id', 'practice_area_id', 'case_number', 'title', 'description',
        'court_name', 'opposing_party', 'opposing_counsel', 'status', 'priority',
        'opened_at', 'closed_at'
    ];

    protected $casts = [
        'opened_at' => 'date',
        'closed_at' => 'date',
    ];

    public function client()
    {
        return $this->belongsTo(Client::class);
    }

    public function practiceArea()
    {
        return $this->belongsTo(PracticeArea::class);
    }

    public function attorneys()
    {
        return $this->belongsToMany(Attorney::class, 'case_attorney')
            ->withPivot('role')
            ->withTimestamps();
    }

    public function hearings()
    {
        return $this->hasMany(Hearing::class);
    }

    public function appointments()
    {
        return $this->hasMany(Appointment::class);
    }

    public function documents()
    {
        return $this->hasMany(Document::class);
    }

    public function invoices()
    {
        return $this->hasMany(Invoice::class);
    }
}
```

## app/Models/Hearing.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Hearing extends Model
{
    use HasFactory;

    protected $fillable = ['legal_case_id', 'title', 'courtroom', 'scheduled_at', 'status', 'notes'];

    protected $casts = ['scheduled_at' => 'datetime'];

    public function legalCase()
    {
        return $this->belongsTo(LegalCase::class);
    }
}
```

## app/Models/Appointment.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Appointment extends Model
{
    use HasFactory;

    protected $fillable = [
        'client_id', 'attorney_id', 'legal_case_id', 'title', 'starts_at',
        'ends_at', 'status', 'location', 'notes'
    ];

    protected $casts = [
        'starts_at' => 'datetime',
        'ends_at' => 'datetime',
    ];

    public function client()
    {
        return $this->belongsTo(Client::class);
    }

    public function attorney()
    {
        return $this->belongsTo(Attorney::class);
    }

    public function legalCase()
    {
        return $this->belongsTo(LegalCase::class);
    }
}
```

## app/Models/Document.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Document extends Model
{
    use HasFactory;

    protected $fillable = ['legal_case_id', 'title', 'document_type', 'file_path', 'description', 'received_at'];

    protected $casts = ['received_at' => 'date'];

    public function legalCase()
    {
        return $this->belongsTo(LegalCase::class);
    }
}
```

## app/Models/Invoice.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Invoice extends Model
{
    use HasFactory;

    protected $fillable = [
        'client_id', 'legal_case_id', 'invoice_number', 'subtotal', 'tax',
        'total', 'status', 'issued_at', 'due_at'
    ];

    protected $casts = [
        'subtotal' => 'decimal:2',
        'tax' => 'decimal:2',
        'total' => 'decimal:2',
        'issued_at' => 'date',
        'due_at' => 'date',
    ];

    public function client()
    {
        return $this->belongsTo(Client::class);
    }

    public function legalCase()
    {
        return $this->belongsTo(LegalCase::class);
    }

    public function payments()
    {
        return $this->hasMany(Payment::class);
    }
}
```

## app/Models/Payment.php

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Payment extends Model
{
    use HasFactory;

    protected $fillable = ['invoice_id', 'amount', 'payment_method', 'reference_number', 'paid_at', 'notes'];

    protected $casts = [
        'amount' => 'decimal:2',
        'paid_at' => 'date',
    ];

    public function invoice()
    {
        return $this->belongsTo(Invoice::class);
    }
}
```

---

# 5. Routes

## routes/web.php

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\ClientController;
use App\Http\Controllers\AttorneyController;
use App\Http\Controllers\PracticeAreaController;
use App\Http\Controllers\LegalCaseController;
use App\Http\Controllers\HearingController;
use App\Http\Controllers\AppointmentController;
use App\Http\Controllers\DocumentController;
use App\Http\Controllers\InvoiceController;
use App\Http\Controllers\PaymentController;

Route::get('/', function () {
    return redirect()->route('legal-cases.index');
});

Route::resource('clients', ClientController::class);
Route::resource('attorneys', AttorneyController::class);
Route::resource('practice-areas', PracticeAreaController::class);
Route::resource('legal-cases', LegalCaseController::class);
Route::resource('hearings', HearingController::class);
Route::resource('appointments', AppointmentController::class);
Route::resource('documents', DocumentController::class);
Route::resource('invoices', InvoiceController::class);
Route::resource('payments', PaymentController::class);
```

---

# 6. Controllers

## app/Http/Controllers/ClientController.php

```php
namespace App\Http\Controllers;

use App\Models\Client;
use Illuminate\Http\Request;

class ClientController extends Controller
{
    public function index()
    {
        $clients = Client::latest()->paginate(10);
        return view('clients.index', compact('clients'));
    }

    public function create()
    {
        return view('clients.create');
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'client_type' => 'required|in:individual,company',
            'first_name' => 'nullable|string|max:255',
            'last_name' => 'nullable|string|max:255',
            'company_name' => 'nullable|string|max:255',
            'email' => 'required|email|unique:clients,email',
            'phone' => 'nullable|string|max:50',
            'address' => 'nullable|string|max:255',
            'city' => 'nullable|string|max:100',
            'state' => 'nullable|string|max:100',
            'postal_code' => 'nullable|string|max:30',
            'country' => 'nullable|string|max:100',
            'notes' => 'nullable|string',
        ]);

        Client::create($data);
        return redirect()->route('clients.index')->with('success', 'Client created successfully.');
    }

    public function show(Client $client)
    {
        $client->load('legalCases', 'appointments', 'invoices');
        return view('clients.show', compact('client'));
    }

    public function edit(Client $client)
    {
        return view('clients.edit', compact('client'));
    }

    public function update(Request $request, Client $client)
    {
        $data = $request->validate([
            'client_type' => 'required|in:individual,company',
            'first_name' => 'nullable|string|max:255',
            'last_name' => 'nullable|string|max:255',
            'company_name' => 'nullable|string|max:255',
            'email' => 'required|email|unique:clients,email,' . $client->id,
            'phone' => 'nullable|string|max:50',
            'address' => 'nullable|string|max:255',
            'city' => 'nullable|string|max:100',
            'state' => 'nullable|string|max:100',
            'postal_code' => 'nullable|string|max:30',
            'country' => 'nullable|string|max:100',
            'notes' => 'nullable|string',
        ]);

        $client->update($data);
        return redirect()->route('clients.index')->with('success', 'Client updated successfully.');
    }

    public function destroy(Client $client)
    {
        $client->delete();
        return redirect()->route('clients.index')->with('success', 'Client deleted successfully.');
    }
}
```

## app/Http/Controllers/AttorneyController.php

```php
namespace App\Http\Controllers;

use App\Models\Attorney;
use Illuminate\Http\Request;

class AttorneyController extends Controller
{
    public function index()
    {
        $attorneys = Attorney::latest()->paginate(10);
        return view('attorneys.index', compact('attorneys'));
    }

    public function create()
    {
        return view('attorneys.create');
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'first_name' => 'required|string|max:255',
            'last_name' => 'required|string|max:255',
            'email' => 'required|email|unique:attorneys,email',
            'phone' => 'nullable|string|max:50',
            'bar_number' => 'nullable|string|max:100|unique:attorneys,bar_number',
            'position' => 'nullable|string|max:100',
            'hourly_rate' => 'required|numeric|min:0',
            'is_active' => 'nullable|boolean',
        ]);

        $data['is_active'] = $request->boolean('is_active');
        Attorney::create($data);

        return redirect()->route('attorneys.index')->with('success', 'Attorney created successfully.');
    }

    public function show(Attorney $attorney)
    {
        $attorney->load('legalCases', 'appointments');
        return view('attorneys.show', compact('attorney'));
    }

    public function edit(Attorney $attorney)
    {
        return view('attorneys.edit', compact('attorney'));
    }

    public function update(Request $request, Attorney $attorney)
    {
        $data = $request->validate([
            'first_name' => 'required|string|max:255',
            'last_name' => 'required|string|max:255',
            'email' => 'required|email|unique:attorneys,email,' . $attorney->id,
            'phone' => 'nullable|string|max:50',
            'bar_number' => 'nullable|string|max:100|unique:attorneys,bar_number,' . $attorney->id,
            'position' => 'nullable|string|max:100',
            'hourly_rate' => 'required|numeric|min:0',
            'is_active' => 'nullable|boolean',
        ]);

        $data['is_active'] = $request->boolean('is_active');
        $attorney->update($data);

        return redirect()->route('attorneys.index')->with('success', 'Attorney updated successfully.');
    }

    public function destroy(Attorney $attorney)
    {
        $attorney->delete();
        return redirect()->route('attorneys.index')->with('success', 'Attorney deleted successfully.');
    }
}
```

## app/Http/Controllers/PracticeAreaController.php

```php
namespace App\Http\Controllers;

use App\Models\PracticeArea;
use Illuminate\Http\Request;

class PracticeAreaController extends Controller
{
    public function index()
    {
        $practiceAreas = PracticeArea::latest()->paginate(10);
        return view('practice-areas.index', compact('practiceAreas'));
    }

    public function create()
    {
        return view('practice-areas.create');
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'name' => 'required|string|max:255|unique:practice_areas,name',
            'description' => 'nullable|string',
        ]);

        PracticeArea::create($data);
        return redirect()->route('practice-areas.index')->with('success', 'Practice area created successfully.');
    }

    public function show(PracticeArea $practiceArea)
    {
        $practiceArea->load('legalCases');
        return view('practice-areas.show', compact('practiceArea'));
    }

    public function edit(PracticeArea $practiceArea)
    {
        return view('practice-areas.edit', compact('practiceArea'));
    }

    public function update(Request $request, PracticeArea $practiceArea)
    {
        $data = $request->validate([
            'name' => 'required|string|max:255|unique:practice_areas,name,' . $practiceArea->id,
            'description' => 'nullable|string',
        ]);

        $practiceArea->update($data);
        return redirect()->route('practice-areas.index')->with('success', 'Practice area updated successfully.');
    }

    public function destroy(PracticeArea $practiceArea)
    {
        $practiceArea->delete();
        return redirect()->route('practice-areas.index')->with('success', 'Practice area deleted successfully.');
    }
}
```

## app/Http/Controllers/LegalCaseController.php

```php
namespace App\Http\Controllers;

use App\Models\Attorney;
use App\Models\Client;
use App\Models\LegalCase;
use App\Models\PracticeArea;
use Illuminate\Http\Request;

class LegalCaseController extends Controller
{
    public function index()
    {
        $legalCases = LegalCase::with('client', 'practiceArea')->latest()->paginate(10);
        return view('legal-cases.index', compact('legalCases'));
    }

    public function create()
    {
        $clients = Client::orderBy('first_name')->get();
        $practiceAreas = PracticeArea::orderBy('name')->get();
        $attorneys = Attorney::where('is_active', true)->orderBy('first_name')->get();

        return view('legal-cases.create', compact('clients', 'practiceAreas', 'attorneys'));
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'client_id' => 'required|exists:clients,id',
            'practice_area_id' => 'nullable|exists:practice_areas,id',
            'case_number' => 'required|string|max:100|unique:legal_cases,case_number',
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'court_name' => 'nullable|string|max:255',
            'opposing_party' => 'nullable|string|max:255',
            'opposing_counsel' => 'nullable|string|max:255',
            'status' => 'required|in:open,pending,closed,archived',
            'priority' => 'required|in:low,medium,high,urgent',
            'opened_at' => 'nullable|date',
            'closed_at' => 'nullable|date',
            'attorney_ids' => 'nullable|array',
            'attorney_ids.*' => 'exists:attorneys,id',
        ]);

        $attorneyIds = $data['attorney_ids'] ?? [];
        unset($data['attorney_ids']);

        $legalCase = LegalCase::create($data);
        $legalCase->attorneys()->sync($attorneyIds);

        return redirect()->route('legal-cases.index')->with('success', 'Case created successfully.');
    }

    public function show(LegalCase $legalCase)
    {
        $legalCase->load('client', 'practiceArea', 'attorneys', 'hearings', 'appointments', 'documents', 'invoices');
        return view('legal-cases.show', compact('legalCase'));
    }

    public function edit(LegalCase $legalCase)
    {
        $clients = Client::orderBy('first_name')->get();
        $practiceAreas = PracticeArea::orderBy('name')->get();
        $attorneys = Attorney::where('is_active', true)->orderBy('first_name')->get();
        $selectedAttorneys = $legalCase->attorneys()->pluck('attorneys.id')->toArray();

        return view('legal-cases.edit', compact('legalCase', 'clients', 'practiceAreas', 'attorneys', 'selectedAttorneys'));
    }

    public function update(Request $request, LegalCase $legalCase)
    {
        $data = $request->validate([
            'client_id' => 'required|exists:clients,id',
            'practice_area_id' => 'nullable|exists:practice_areas,id',
            'case_number' => 'required|string|max:100|unique:legal_cases,case_number,' . $legalCase->id,
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'court_name' => 'nullable|string|max:255',
            'opposing_party' => 'nullable|string|max:255',
            'opposing_counsel' => 'nullable|string|max:255',
            'status' => 'required|in:open,pending,closed,archived',
            'priority' => 'required|in:low,medium,high,urgent',
            'opened_at' => 'nullable|date',
            'closed_at' => 'nullable|date',
            'attorney_ids' => 'nullable|array',
            'attorney_ids.*' => 'exists:attorneys,id',
        ]);

        $attorneyIds = $data['attorney_ids'] ?? [];
        unset($data['attorney_ids']);

        $legalCase->update($data);
        $legalCase->attorneys()->sync($attorneyIds);

        return redirect()->route('legal-cases.index')->with('success', 'Case updated successfully.');
    }

    public function destroy(LegalCase $legalCase)
    {
        $legalCase->delete();
        return redirect()->route('legal-cases.index')->with('success', 'Case deleted successfully.');
    }
}
```

## Generic Controllers for Hearing, Appointment, Document, Invoice, Payment

Use the same pattern as above. Below are complete versions.

## app/Http/Controllers/HearingController.php

```php
namespace App\Http\Controllers;

use App\Models\Hearing;
use App\Models\LegalCase;
use Illuminate\Http\Request;

class HearingController extends Controller
{
    public function index()
    {
        $hearings = Hearing::with('legalCase')->latest()->paginate(10);
        return view('hearings.index', compact('hearings'));
    }

    public function create()
    {
        $legalCases = LegalCase::orderBy('title')->get();
        return view('hearings.create', compact('legalCases'));
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'legal_case_id' => 'required|exists:legal_cases,id',
            'title' => 'required|string|max:255',
            'courtroom' => 'nullable|string|max:255',
            'scheduled_at' => 'required|date',
            'status' => 'required|in:scheduled,completed,adjourned,cancelled',
            'notes' => 'nullable|string',
        ]);

        Hearing::create($data);
        return redirect()->route('hearings.index')->with('success', 'Hearing created successfully.');
    }

    public function show(Hearing $hearing)
    {
        $hearing->load('legalCase');
        return view('hearings.show', compact('hearing'));
    }

    public function edit(Hearing $hearing)
    {
        $legalCases = LegalCase::orderBy('title')->get();
        return view('hearings.edit', compact('hearing', 'legalCases'));
    }

    public function update(Request $request, Hearing $hearing)
    {
        $data = $request->validate([
            'legal_case_id' => 'required|exists:legal_cases,id',
            'title' => 'required|string|max:255',
            'courtroom' => 'nullable|string|max:255',
            'scheduled_at' => 'required|date',
            'status' => 'required|in:scheduled,completed,adjourned,cancelled',
            'notes' => 'nullable|string',
        ]);

        $hearing->update($data);
        return redirect()->route('hearings.index')->with('success', 'Hearing updated successfully.');
    }

    public function destroy(Hearing $hearing)
    {
        $hearing->delete();
        return redirect()->route('hearings.index')->with('success', 'Hearing deleted successfully.');
    }
}
```

## app/Http/Controllers/AppointmentController.php

```php
namespace App\Http\Controllers;

use App\Models\Appointment;
use App\Models\Attorney;
use App\Models\Client;
use App\Models\LegalCase;
use Illuminate\Http\Request;

class AppointmentController extends Controller
{
    public function index()
    {
        $appointments = Appointment::with('client', 'attorney', 'legalCase')->latest()->paginate(10);
        return view('appointments.index', compact('appointments'));
    }

    public function create()
    {
        $clients = Client::orderBy('first_name')->get();
        $attorneys = Attorney::where('is_active', true)->orderBy('first_name')->get();
        $legalCases = LegalCase::orderBy('title')->get();
        return view('appointments.create', compact('clients', 'attorneys', 'legalCases'));
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'client_id' => 'required|exists:clients,id',
            'attorney_id' => 'nullable|exists:attorneys,id',
            'legal_case_id' => 'nullable|exists:legal_cases,id',
            'title' => 'required|string|max:255',
            'starts_at' => 'required|date',
            'ends_at' => 'nullable|date|after_or_equal:starts_at',
            'status' => 'required|in:scheduled,completed,cancelled,no_show',
            'location' => 'nullable|string|max:255',
            'notes' => 'nullable|string',
        ]);

        Appointment::create($data);
        return redirect()->route('appointments.index')->with('success', 'Appointment created successfully.');
    }

    public function show(Appointment $appointment)
    {
        $appointment->load('client', 'attorney', 'legalCase');
        return view('appointments.show', compact('appointment'));
    }

    public function edit(Appointment $appointment)
    {
        $clients = Client::orderBy('first_name')->get();
        $attorneys = Attorney::where('is_active', true)->orderBy('first_name')->get();
        $legalCases = LegalCase::orderBy('title')->get();
        return view('appointments.edit', compact('appointment', 'clients', 'attorneys', 'legalCases'));
    }

    public function update(Request $request, Appointment $appointment)
    {
        $data = $request->validate([
            'client_id' => 'required|exists:clients,id',
            'attorney_id' => 'nullable|exists:attorneys,id',
            'legal_case_id' => 'nullable|exists:legal_cases,id',
            'title' => 'required|string|max:255',
            'starts_at' => 'required|date',
            'ends_at' => 'nullable|date|after_or_equal:starts_at',
            'status' => 'required|in:scheduled,completed,cancelled,no_show',
            'location' => 'nullable|string|max:255',
            'notes' => 'nullable|string',
        ]);

        $appointment->update($data);
        return redirect()->route('appointments.index')->with('success', 'Appointment updated successfully.');
    }

    public function destroy(Appointment $appointment)
    {
        $appointment->delete();
        return redirect()->route('appointments.index')->with('success', 'Appointment deleted successfully.');
    }
}
```

## app/Http/Controllers/DocumentController.php

```php
namespace App\Http\Controllers;

use App\Models\Document;
use App\Models\LegalCase;
use Illuminate\Http\Request;

class DocumentController extends Controller
{
    public function index()
    {
        $documents = Document::with('legalCase')->latest()->paginate(10);
        return view('documents.index', compact('documents'));
    }

    public function create()
    {
        $legalCases = LegalCase::orderBy('title')->get();
        return view('documents.create', compact('legalCases'));
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'legal_case_id' => 'required|exists:legal_cases,id',
            'title' => 'required|string|max:255',
            'document_type' => 'nullable|string|max:100',
            'file_path' => 'nullable|string|max:255',
            'description' => 'nullable|string',
            'received_at' => 'nullable|date',
        ]);

        Document::create($data);
        return redirect()->route('documents.index')->with('success', 'Document created successfully.');
    }

    public function show(Document $document)
    {
        $document->load('legalCase');
        return view('documents.show', compact('document'));
    }

    public function edit(Document $document)
    {
        $legalCases = LegalCase::orderBy('title')->get();
        return view('documents.edit', compact('document', 'legalCases'));
    }

    public function update(Request $request, Document $document)
    {
        $data = $request->validate([
            'legal_case_id' => 'required|exists:legal_cases,id',
            'title' => 'required|string|max:255',
            'document_type' => 'nullable|string|max:100',
            'file_path' => 'nullable|string|max:255',
            'description' => 'nullable|string',
            'received_at' => 'nullable|date',
        ]);

        $document->update($data);
        return redirect()->route('documents.index')->with('success', 'Document updated successfully.');
    }

    public function destroy(Document $document)
    {
        $document->delete();
        return redirect()->route('documents.index')->with('success', 'Document deleted successfully.');
    }
}
```

## app/Http/Controllers/InvoiceController.php

```php
namespace App\Http\Controllers;

use App\Models\Client;
use App\Models\Invoice;
use App\Models\LegalCase;
use Illuminate\Http\Request;

class InvoiceController extends Controller
{
    public function index()
    {
        $invoices = Invoice::with('client', 'legalCase')->latest()->paginate(10);
        return view('invoices.index', compact('invoices'));
    }

    public function create()
    {
        $clients = Client::orderBy('first_name')->get();
        $legalCases = LegalCase::orderBy('title')->get();
        return view('invoices.create', compact('clients', 'legalCases'));
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'client_id' => 'required|exists:clients,id',
            'legal_case_id' => 'nullable|exists:legal_cases,id',
            'invoice_number' => 'required|string|max:100|unique:invoices,invoice_number',
            'subtotal' => 'required|numeric|min:0',
            'tax' => 'required|numeric|min:0',
            'total' => 'required|numeric|min:0',
            'status' => 'required|in:draft,sent,paid,overdue,cancelled',
            'issued_at' => 'nullable|date',
            'due_at' => 'nullable|date',
        ]);

        Invoice::create($data);
        return redirect()->route('invoices.index')->with('success', 'Invoice created successfully.');
    }

    public function show(Invoice $invoice)
    {
        $invoice->load('client', 'legalCase', 'payments');
        return view('invoices.show', compact('invoice'));
    }

    public function edit(Invoice $invoice)
    {
        $clients = Client::orderBy('first_name')->get();
        $legalCases = LegalCase::orderBy('title')->get();
        return view('invoices.edit', compact('invoice', 'clients', 'legalCases'));
    }

    public function update(Request $request, Invoice $invoice)
    {
        $data = $request->validate([
            'client_id' => 'required|exists:clients,id',
            'legal_case_id' => 'nullable|exists:legal_cases,id',
            'invoice_number' => 'required|string|max:100|unique:invoices,invoice_number,' . $invoice->id,
            'subtotal' => 'required|numeric|min:0',
            'tax' => 'required|numeric|min:0',
            'total' => 'required|numeric|min:0',
            'status' => 'required|in:draft,sent,paid,overdue,cancelled',
            'issued_at' => 'nullable|date',
            'due_at' => 'nullable|date',
        ]);

        $invoice->update($data);
        return redirect()->route('invoices.index')->with('success', 'Invoice updated successfully.');
    }

    public function destroy(Invoice $invoice)
    {
        $invoice->delete();
        return redirect()->route('invoices.index')->with('success', 'Invoice deleted successfully.');
    }
}
```

## app/Http/Controllers/PaymentController.php

```php
namespace App\Http\Controllers;

use App\Models\Invoice;
use App\Models\Payment;
use Illuminate\Http\Request;

class PaymentController extends Controller
{
    public function index()
    {
        $payments = Payment::with('invoice.client')->latest()->paginate(10);
        return view('payments.index', compact('payments'));
    }

    public function create()
    {
        $invoices = Invoice::with('client')->orderBy('invoice_number')->get();
        return view('payments.create', compact('invoices'));
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'invoice_id' => 'required|exists:invoices,id',
            'amount' => 'required|numeric|min:0.01',
            'payment_method' => 'nullable|string|max:100',
            'reference_number' => 'nullable|string|max:100',
            'paid_at' => 'required|date',
            'notes' => 'nullable|string',
        ]);

        Payment::create($data);
        return redirect()->route('payments.index')->with('success', 'Payment created successfully.');
    }

    public function show(Payment $payment)
    {
        $payment->load('invoice.client');
        return view('payments.show', compact('payment'));
    }

    public function edit(Payment $payment)
    {
        $invoices = Invoice::with('client')->orderBy('invoice_number')->get();
        return view('payments.edit', compact('payment', 'invoices'));
    }

    public function update(Request $request, Payment $payment)
    {
        $data = $request->validate([
            'invoice_id' => 'required|exists:invoices,id',
            'amount' => 'required|numeric|min:0.01',
            'payment_method' => 'nullable|string|max:100',
            'reference_number' => 'nullable|string|max:100',
            'paid_at' => 'required|date',
            'notes' => 'nullable|string',
        ]);

        $payment->update($data);
        return redirect()->route('payments.index')->with('success', 'Payment updated successfully.');
    }

    public function destroy(Payment $payment)
    {
        $payment->delete();
        return redirect()->route('payments.index')->with('success', 'Payment deleted successfully.');
    }
}
```

---

# 7. Shared Blade Layout

## resources/views/layouts/app.blade.php

```blade
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Law Firm Management</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
    <div class="container">
        <a class="navbar-brand" href="{{ route('legal-cases.index') }}">Law Firm</a>
        <div class="navbar-nav">
            <a class="nav-link" href="{{ route('clients.index') }}">Clients</a>
            <a class="nav-link" href="{{ route('attorneys.index') }}">Attorneys</a>
            <a class="nav-link" href="{{ route('practice-areas.index') }}">Practice Areas</a>
            <a class="nav-link" href="{{ route('legal-cases.index') }}">Cases</a>
            <a class="nav-link" href="{{ route('hearings.index') }}">Hearings</a>
            <a class="nav-link" href="{{ route('appointments.index') }}">Appointments</a>
            <a class="nav-link" href="{{ route('documents.index') }}">Documents</a>
            <a class="nav-link" href="{{ route('invoices.index') }}">Invoices</a>
            <a class="nav-link" href="{{ route('payments.index') }}">Payments</a>
        </div>
    </div>
</nav>

<main class="container">
    @if(session('success'))
        <div class="alert alert-success">{{ session('success') }}</div>
    @endif

    @if($errors->any())
        <div class="alert alert-danger">
            <ul class="mb-0">
                @foreach($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    @yield('content')
</main>
</body>
</html>
```

---

# 8. Blade Views — Clients

Create folder:

```bash
mkdir -p resources/views/clients
```

## resources/views/clients/index.blade.php

```blade
@extends('layouts.app')

@section('content')
<div class="d-flex justify-content-between align-items-center mb-3">
    <h1>Clients</h1>
    <a href="{{ route('clients.create') }}" class="btn btn-primary">Add Client</a>
</div>

<table class="table table-bordered table-striped">
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Email</th>
            <th>Phone</th>
            <th width="220">Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach($clients as $client)
            <tr>
                <td>{{ $client->display_name }}</td>
                <td>{{ ucfirst($client->client_type) }}</td>
                <td>{{ $client->email }}</td>
                <td>{{ $client->phone }}</td>
                <td>
                    <a href="{{ route('clients.show', $client) }}" class="btn btn-sm btn-info">View</a>
                    <a href="{{ route('clients.edit', $client) }}" class="btn btn-sm btn-warning">Edit</a>
                    <form action="{{ route('clients.destroy', $client) }}" method="POST" class="d-inline">
                        @csrf
                        @method('DELETE')
                        <button class="btn btn-sm btn-danger" onclick="return confirm('Delete this client?')">Delete</button>
                    </form>
                </td>
            </tr>
        @endforeach
    </tbody>
</table>

{{ $clients->links() }}
@endsection
```

## resources/views/clients/_form.blade.php

```blade
@csrf

<div class="mb-3">
    <label class="form-label">Client Type</label>
    <select name="client_type" class="form-control" required>
        <option value="individual" @selected(old('client_type', $client->client_type ?? '') === 'individual')>Individual</option>
        <option value="company" @selected(old('client_type', $client->client_type ?? '') === 'company')>Company</option>
    </select>
</div>

<div class="row">
    <div class="col-md-6 mb-3">
        <label class="form-label">First Name</label>
        <input type="text" name="first_name" class="form-control" value="{{ old('first_name', $client->first_name ?? '') }}">
    </div>
    <div class="col-md-6 mb-3">
        <label class="form-label">Last Name</label>
        <input type="text" name="last_name" class="form-control" value="{{ old('last_name', $client->last_name ?? '') }}">
    </div>
</div>

<div class="mb-3">
    <label class="form-label">Company Name</label>
    <input type="text" name="company_name" class="form-control" value="{{ old('company_name', $client->company_name ?? '') }}">
</div>

<div class="row">
    <div class="col-md-6 mb-3">
        <label class="form-label">Email</label>
        <input type="email" name="email" class="form-control" value="{{ old('email', $client->email ?? '') }}" required>
    </div>
    <div class="col-md-6 mb-3">
        <label class="form-label">Phone</label>
        <input type="text" name="phone" class="form-control" value="{{ old('phone', $client->phone ?? '') }}">
    </div>
</div>

<div class="mb-3">
    <label class="form-label">Address</label>
    <input type="text" name="address" class="form-control" value="{{ old('address', $client->address ?? '') }}">
</div>

<div class="row">
    <div class="col-md-3 mb-3">
        <label class="form-label">City</label>
        <input type="text" name="city" class="form-control" value="{{ old('city', $client->city ?? '') }}">
    </div>
    <div class="col-md-3 mb-3">
        <label class="form-label">State</label>
        <input type="text" name="state" class="form-control" value="{{ old('state', $client->state ?? '') }}">
    </div>
    <div class="col-md-3 mb-3">
        <label class="form-label">Postal Code</label>
        <input type="text" name="postal_code" class="form-control" value="{{ old('postal_code', $client->postal_code ?? '') }}">
    </div>
    <div class="col-md-3 mb-3">
        <label class="form-label">Country</label>
        <input type="text" name="country" class="form-control" value="{{ old('country', $client->country ?? '') }}">
    </div>
</div>

<div class="mb-3">
    <label class="form-label">Notes</label>
    <textarea name="notes" class="form-control">{{ old('notes', $client->notes ?? '') }}</textarea>
</div>

<button class="btn btn-success">Save</button>
<a href="{{ route('clients.index') }}" class="btn btn-secondary">Cancel</a>
```

## resources/views/clients/create.blade.php

```blade
@extends('layouts.app')

@section('content')
<h1>Add Client</h1>
<form action="{{ route('clients.store') }}" method="POST">
    @include('clients._form')
</form>
@endsection
```

## resources/views/clients/edit.blade.php

```blade
@extends('layouts.app')

@section('content')
<h1>Edit Client</h1>
<form action="{{ route('clients.update', $client) }}" method="POST">
    @method('PUT')
    @include('clients._form')
</form>
@endsection
```

## resources/views/clients/show.blade.php

```blade
@extends('layouts.app')

@section('content')
<h1>{{ $client->display_name }}</h1>

<div class="card mb-3">
    <div class="card-body">
        <p><strong>Type:</strong> {{ ucfirst($client->client_type) }}</p>
        <p><strong>Email:</strong> {{ $client->email }}</p>
        <p><strong>Phone:</strong> {{ $client->phone }}</p>
        <p><strong>Address:</strong> {{ $client->address }}</p>
        <p><strong>Notes:</strong> {{ $client->notes }}</p>
    </div>
</div>

<a href="{{ route('clients.edit', $client) }}" class="btn btn-warning">Edit</a>
<a href="{{ route('clients.index') }}" class="btn btn-secondary">Back</a>
@endsection
```

---

# 9. Blade Views — Legal Cases

Create folder:

```bash
mkdir -p resources/views/legal-cases
```

## resources/views/legal-cases/index.blade.php

```blade
@extends('layouts.app')

@section('content')
<div class="d-flex justify-content-between align-items-center mb-3">
    <h1>Legal Cases</h1>
    <a href="{{ route('legal-cases.create') }}" class="btn btn-primary">Add Case</a>
</div>

<table class="table table-bordered table-striped">
    <thead>
        <tr>
            <th>Case #</th>
            <th>Title</th>
            <th>Client</th>
            <th>Practice Area</th>
            <th>Status</th>
            <th>Priority</th>
            <th width="220">Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach($legalCases as $case)
            <tr>
                <td>{{ $case->case_number }}</td>
                <td>{{ $case->title }}</td>
                <td>{{ $case->client?->display_name }}</td>
                <td>{{ $case->practiceArea?->name }}</td>
                <td>{{ ucfirst($case->status) }}</td>
                <td>{{ ucfirst($case->priority) }}</td>
                <td>
                    <a href="{{ route('legal-cases.show', $case) }}" class="btn btn-sm btn-info">View</a>
                    <a href="{{ route('legal-cases.edit', $case) }}" class="btn btn-sm btn-warning">Edit</a>
                    <form action="{{ route('legal-cases.destroy', $case) }}" method="POST" class="d-inline">
                        @csrf
                        @method('DELETE')
                        <button class="btn btn-sm btn-danger" onclick="return confirm('Delete this case?')">Delete</button>
                    </form>
                </td>
            </tr>
        @endforeach
    </tbody>
</table>

{{ $legalCases->links() }}
@endsection
```

## resources/views/legal-cases/_form.blade.php

```blade
@csrf

<div class="row">
    <div class="col-md-6 mb-3">
        <label class="form-label">Client</label>
        <select name="client_id" class="form-control" required>
            <option value="">Select client</option>
            @foreach($clients as $client)
                <option value="{{ $client->id }}" @selected(old('client_id', $legalCase->client_id ?? '') == $client->id)>
                    {{ $client->display_name }}
                </option>
            @endforeach
        </select>
    </div>
    <div class="col-md-6 mb-3">
        <label class="form-label">Practice Area</label>
        <select name="practice_area_id" class="form-control">
            <option value="">Select practice area</option>
            @foreach($practiceAreas as $area)
                <option value="{{ $area->id }}" @selected(old('practice_area_id', $legalCase->practice_area_id ?? '') == $area->id)>
                    {{ $area->name }}
                </option>
            @endforeach
        </select>
    </div>
</div>

<div class="row">
    <div class="col-md-4 mb-3">
        <label class="form-label">Case Number</label>
        <input type="text" name="case_number" class="form-control" value="{{ old('case_number', $legalCase->case_number ?? '') }}" required>
    </div>
    <div class="col-md-8 mb-3">
        <label class="form-label">Title</label>
        <input type="text" name="title" class="form-control" value="{{ old('title', $legalCase->title ?? '') }}" required>
    </div>
</div>

<div class="mb-3">
    <label class="form-label">Description</label>
    <textarea name="description" class="form-control">{{ old('description', $legalCase->description ?? '') }}</textarea>
</div>

<div class="row">
    <div class="col-md-4 mb-3">
        <label class="form-label">Court Name</label>
        <input type="text" name="court_name" class="form-control" value="{{ old('court_name', $legalCase->court_name ?? '') }}">
    </div>
    <div class="col-md-4 mb-3">
        <label class="form-label">Opposing Party</label>
        <input type="text" name="opposing_party" class="form-control" value="{{ old('opposing_party', $legalCase->opposing_party ?? '') }}">
    </div>
    <div class="col-md-4 mb-3">
        <label class="form-label">Opposing Counsel</label>
        <input type="text" name="opposing_counsel" class="form-control" value="{{ old('opposing_counsel', $legalCase->opposing_counsel ?? '') }}">
    </div>
</div>

<div class="row">
    <div class="col-md-3 mb-3">
        <label class="form-label">Status</label>
        <select name="status" class="form-control" required>
            @foreach(['open', 'pending', 'closed', 'archived'] as $status)
                <option value="{{ $status }}" @selected(old('status', $legalCase->status ?? 'open') === $status)>{{ ucfirst($status) }}</option>
            @endforeach
        </select>
    </div>
    <div class="col-md-3 mb-3">
        <label class="form-label">Priority</label>
        <select name="priority" class="form-control" required>
            @foreach(['low', 'medium', 'high', 'urgent'] as $priority)
                <option value="{{ $priority }}" @selected(old('priority', $legalCase->priority ?? 'medium') === $priority)>{{ ucfirst($priority) }}</option>
            @endforeach
        </select>
    </div>
    <div class="col-md-3 mb-3">
        <label class="form-label">Opened At</label>
        <input type="date" name="opened_at" class="form-control" value="{{ old('opened_at', isset($legalCase) && $legalCase->opened_at ? $legalCase->opened_at->format('Y-m-d') : '') }}">
    </div>
    <div class="col-md-3 mb-3">
        <label class="form-label">Closed At</label>
        <input type="date" name="closed_at" class="form-control" value="{{ old('closed_at', isset($legalCase) && $legalCase->closed_at ? $legalCase->closed_at->format('Y-m-d') : '') }}">
    </div>
</div>

<div class="mb-3">
    <label class="form-label">Assigned Attorneys</label>
    <select name="attorney_ids[]" class="form-control" multiple>
        @foreach($attorneys as $attorney)
            <option value="{{ $attorney->id }}" @selected(in_array($attorney->id, old('attorney_ids', $selectedAttorneys ?? [])))>
                {{ $attorney->full_name }}
            </option>
        @endforeach
    </select>
</div>

<button class="btn btn-success">Save</button>
<a href="{{ route('legal-cases.index') }}" class="btn btn-secondary">Cancel</a>
```

## resources/views/legal-cases/create.blade.php

```blade
@extends('layouts.app')

@section('content')
<h1>Add Case</h1>
<form action="{{ route('legal-cases.store') }}" method="POST">
    @include('legal-cases._form')
</form>
@endsection
```

## resources/views/legal-cases/edit.blade.php

```blade
@extends('layouts.app')

@section('content')
<h1>Edit Case</h1>
<form action="{{ route('legal-cases.update', $legalCase) }}" method="POST">
    @method('PUT')
    @include('legal-cases._form')
</form>
@endsection
```

## resources/views/legal-cases/show.blade.php

```blade
@extends('layouts.app')

@section('content')
<h1>{{ $legalCase->title }}</h1>

<div class="card mb-3">
    <div class="card-body">
        <p><strong>Case Number:</strong> {{ $legalCase->case_number }}</p>
        <p><strong>Client:</strong> {{ $legalCase->client?->display_name }}</p>
        <p><strong>Practice Area:</strong> {{ $legalCase->practiceArea?->name }}</p>
        <p><strong>Status:</strong> {{ ucfirst($legalCase->status) }}</p>
        <p><strong>Priority:</strong> {{ ucfirst($legalCase->priority) }}</p>
        <p><strong>Court:</strong> {{ $legalCase->court_name }}</p>
        <p><strong>Opposing Party:</strong> {{ $legalCase->opposing_party }}</p>
        <p><strong>Description:</strong> {{ $legalCase->description }}</p>
    </div>
</div>

<h3>Assigned Attorneys</h3>
<ul>
    @foreach($legalCase->attorneys as $attorney)
        <li>{{ $attorney->full_name }}</li>
    @endforeach
</ul>

<a href="{{ route('legal-cases.edit', $legalCase) }}" class="btn btn-warning">Edit</a>
<a href="{{ route('legal-cases.index') }}" class="btn btn-secondary">Back</a>
@endsection
```

---

# 10. Reusable Simple Views for Smaller Modules

For `attorneys`, `practice-areas`, `hearings`, `appointments`, `documents`, `invoices`, and `payments`, use the same Blade pattern as clients and legal-cases:

* `index.blade.php`
* `create.blade.php`
* `edit.blade.php`
* `show.blade.php`
* `_form.blade.php`

The controller data already supports those views.

---

# 11. Suggested Folder Structure

```text
resources/views/
├── layouts/
│   └── app.blade.php
├── clients/
├── attorneys/
├── practice-areas/
├── legal-cases/
├── hearings/
├── appointments/
├── documents/
├── invoices/
└── payments/
```

---

# 12. Recommended Enhancements

For a production-grade attorney platform, add:

* Laravel Breeze or Jetstream authentication
* Role-based permissions: admin, attorney, paralegal, billing staff
* Document upload using Laravel storage
* Audit log for sensitive case changes
* Conflict-checking before client/case creation
* Invoice line items instead of only subtotal/tax/total
* Soft deletes for clients and cases
* Search and filtering by status, attorney, court, and hearing date
* Email reminders for appointments and hearings
* Secure access controls per case

---

# 13. GitHub Push Commands

```bash
git init
git add .
git commit -m "Add law firm CRUD system"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/attorney-web-app.git
git push -u origin main
```
