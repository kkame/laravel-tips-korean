# 라라벨 팁 한국어

라라벨 개발자들을 위한 멋진 Laravel 팁과 트릭 모음입니다. PR 및 아이디어를 환영합니다! <br>이 문서는 [PovilasKorop](https://github.com/PovilasKorop) 과 [MarceauKa](https://github.com/MarceauKa) 의 아이디어를 번역한 문서입니다.

**2021 년 1 월 29 일 업데이트** : 현재 **125 개의 팁** 이 14 개 섹션으로 나뉩니다.

## 목차

- [DB 모델 및 Eloquent](#db-models-and-eloquent) (31 개)
- [모델 관계](#models-relations) (22 개)
- [마이그레이션](#migrations) (8 개)
- [뷰](#views) (8 개)
- [라우팅](#routing) (13 개)
- [유효성 검사](#validation) (7 개)
- [컬렉션](#collections) (4 개)
- [인증](#auth) (5 개)
- [메일](#mail) (4 개)
- [Artisan](#artisan) (5 개)
- [팩토리](#factories) (2 개)
- [로그 및 디버그](#log-and-debug) (2 개)
- [API](#api) (2 개)
- [기타](#other) (12 개)

## DB 모델 및 Eloquent <a id="db-models-and-eloquent"></a>

⬆️ [맨 위로 이동](#laravel-tips) ➡️ [다음 (모델 관계)](#models-relations)

- [Eloquent 날짜 관련 검색 메서드](#eloquent-where-date-methods)
- [값의 증가 및 감소](#increments-and-decrements)
- [타임 스탬프 컬럼이 없을 때](#no-timestamp-columns)
- [옵저버로 로그인 한 사용자 지정](#set-logged-in-user-with-observers)
- [소프트 삭제 : 다중 복원](#soft-deletes-multiple-restore)
- [Model::all의 컬럼 지정](#model-all-columns)
- [실패하거나 실패하지 않기](#to-fail-or-not-to-fail)
- [열 이름 변경](#column-name-change)
- [쿼리 결과에 대한 개별 수정](#map-query-results)
- [기본 타임 스탬프 필드 변경](#change-default-timestamp-fields)
- [created_at을 손쉽게 정렬하기](#quick-order-by-created_at)
- [레코드 생성시 자동으로 생성되는 열 값](#automatic-column-value-when-creating-records)
- [DB Raw 쿼리 계산을 더 빠르게](#db-raw-query-calculations-run-faster)
- [한개 이상의 스코프](#more-than-one-scope)
- [카본을 전환해줄 필요가 없습니다](#no-need-to-convert-carbon)
- [첫 글자로 그룹화](#grouping-by-first-letter)
- [열 업데이트 안 함](#never-update-the-column)
- [여러개를 찾기](#find-many)
- [Key로 찾기](#find-by-key)
- [자동 증가 대신 UUID 사용](#use-uuid-instead-of-auto-increment)
- [Laravel에서 sub select 사용법](#sub-selects-in-laravel-way)
- [일부 열 숨기기](#hide-some-columns)
- [정확한 DB 오류](#exact-db-error)
- [쿼리 빌더에서의 소프트 삭제](#soft-deletes-with-query-builder)
- [올바른 오래된 SQL 쿼리](#good-old-sql-query)
- [DB 트랜잭션 사용](#use-db-transactions)
- [업데이트 또는 생성](#update-or-create)
- [저장시 캐시 삭제](#forget-cache-on-save)
- [Created_at 및 Updated_at 형식 변경](#change-format-of-created_at-and-updated_at)
- [JSON에 배열 유형 저장](#storing-array-type-into-json)
- [모델 사본 만들기](#make-a-copy-of-the-model)

### Eloquent 날짜 관련 검색 메서드 <a id="eloquent-where-date-methods"></a>

`whereDay()` , `whereMonth()` , `whereYear()` , `whereDate()` 및 `whereTime()` 함수로 날짜를 검색하십시오.

```php
$products = Product::whereDate('created_at', '2018-01-31')->get();
$products = Product::whereMonth('created_at', '12')->get();
$products = Product::whereDay('created_at', '31')->get();
$products = Product::whereYear('created_at', date('Y'))->get();
$products = Product::whereTime('created_at', '=', '14:13:58')->get();
```

### 값의 증가 및 감소 <a id="increments-and-decrements"></a>

일부 테이블에서 일부 DB 컬럼의 값을 증가 시키려면 `increment()` 함수를 사용하십시오. 아, 그리고 당신은 1만큼 증가시킬 수있을뿐만 아니라 50과 같은 어떤 숫자도 증가시킬 수 있습니다.

```php
Post::find($post_id)->increment('view_count');
User::find($user_id)->increment('points', 50);
```

### 타임 스탬프 컬럼이 없을 때 <a id="no-timestamp-columns"></a>

DB 테이블에 타임 스탬프 필드 `created_at` 및 `updated_at`가 포함되어 있지 않은 경우, `$timestamps = false`를 사용하여 Eloquent 모델이 이 속성을 사용하지 않도록 지정할 수 있습니다.

```php
class Company extends Model
{
    public $timestamps = false;
}
```

### 옵저버로 로그인 한 사용자 지정 <a id="set-logged-in-user-with-observers"></a>

현재 로그인 한 사용자를 `user_id` 필드에 자동으로 지정되게하려면 `make:observer` 를 사용하고 `creating()` 메서드를 추가합니다.

```php
class PostObserver
{
    public function creating(Post $post)
    {
        $post->user_id = auth()->id();
    }
}
```

### 소프트 삭제 : 다중 복원 <a id="soft-deletes-multiple-restore"></a>

소프트 삭제를 사용하면 한 문장으로 여러 행을 복원 할 수 있습니다.

```php
Post::withTrashed()->where('author_id', 1)->restore();
```

### Model::all의 컬럼 지정 <a id="model-all-columns"></a>

Eloquent의 `Model::all()` 호출 할 때 반환 할 열을 지정할 수 있습니다.

```php
$users = User::all(['id', 'name', 'email']);
```

### 실패하거나 실패하지 않기 <a id="to-fail-or-not-to-fail"></a>

`findOrFail()` 외에도 쿼리에 대한 레코드가없는 경우 404 페이지를 반환하는 Eloquent 메서드 `firstOrFail()`도 있습니다.

```php
$user = User::where('email', 'povilas@laraveldaily.com')->firstOrFail();
```

### 열 이름 변경 <a id="column-name-change"></a>

Eloquent Query Builder에서 "as"를 지정하여 일반 SQL 쿼리처럼 다른 이름을 가진 열을 반환 할 수 있습니다.

```php
$users = DB::table('users')->select('name', 'email as user_email')->get();
```

### 쿼리 결과에 대한 개별 수정 <a id="map-query-results"></a>

Eloquent 쿼리 후에 Collections에서 `map()` 함수를 사용하여 행을 수정할 수 있습니다.

```php
$users = User::where('role_id', 1)->get()->map(function (User $user) {
    $user->some_column = some_function($user);
    return $user;
});
```

### 기본 타임 스탬프 필드 변경 <a id="change-default-timestamp-fields"></a>

Laravel이 아닌 데이터베이스로 작업하고 있고 타임스탬프 열의 이름이 다르다면 어떻게 해야할까요? 아마도 create_time 이나 update_time 같은 열이 있을 겁니다. 다행히도 모델에서 변경 할 수 있습니다.

```php
class Role extends Model
{
    const CREATED_AT = 'create_time';
    const UPDATED_AT = 'update_time';
}
```

### created_at을 손쉽게 정렬하기 <a id="quick-order-by-created_at"></a>

이것을

```php
User::orderBy('created_at', 'desc')->get();
```

더 쉽게 할 수 있습니다.

```php
User::latest()->get();
```

기본적으로 `latest()` 는 `created_at`을 이용하여 정렬합니다.

`created_at` 오름차순으로 정렬하는, 반대 메소드 `oldest()`도 있습니다.

```php
User::oldest()->get();
```

또한 정렬할 열을 바꿀 수도 있습니다. 예를 들어 `updated_at` 를 사용하려면 아래처럼 사용하면 됩니다.

```php
$lastUpdatedUser = User::latest('updated_at')->first();
```

### 레코드 생성시 자동으로 생성되는 열 값 <a id="automatic-column-value-when-creating-records"></a>

레코드 생성시 DB 컬럼 값을 생성하려면 모델의 `boot()` 메소드에 추가합니다. 예를 들어 "position"필드가 있고 사용 가능한 다음 위치를 새 레코드에 할당하려면 (예 `Country::max('position') + 1)` 과 같이하십시오.

```php
class Country extends Model {
    protected static function boot()
    {
        parent::boot();

        Country::creating(function($model) {
            $model->position = Country::max('position') + 1;
        });
    }
}
```

### DB Raw 쿼리 계산을 더 빠르게 <a id="db-raw-query-calculations-run-faster"></a>

`whereRaw()` 메서드와 같은 SQL 원시 쿼리를 사용하여 일부 DB 특정 계산을 Laravel이 아닌 쿼리에서 직접 수행하면 일반적으로 결과가 더 빠릅니다. 예를 들어, 등록 후 30 일 이상 활성 상태였던 사용자를 얻으려면 다음 코드를 사용하십시오. (역자: 단 이 경우는 ORM의 장점인 추상화의 이점을 얻지 못하고 특정 데이터베이스에 종속되는 사이드이펙트가 생길 수 있습니다. 이 부분을 감안하고 사용하시기 바랍니다. 이것은 사용하지 말라는 의미가 아닌 주의를 필요로 하는 의미로 남기는 글입니다.)

```php
User::where('active', 1)
    ->whereRaw('TIMESTAMPDIFF(DAY, created_at, updated_at) > ?', 30)
    ->get();
```

### 한개 이상의 스코프 <a id="more-than-one-scope"></a>

쿼리에서 하나 이상의 스코프를 사용하여 Eloquent에서 쿼리 스코프를 결합하고 연결할 수 있습니다.

모델:

```php
public function scopeActive($query) {
    return $query->where('active', 1);
}

public function scopeRegisteredWithinDays($query, $days) {
    return $query->where('created_at', '>=', now()->subDays($days));
}
```

일부 컨트롤러 :

```php
$users = User::registeredWithinDays(30)->active()->get();
```

### 카본을 전환해줄 필요가 없습니다 <a id="no-need-to-convert-carbon"></a>

`whereDate()`를 이용해 오늘 기준의 레코드를 조회할 경우 Carbon의 `now()`를 사용할 수 있으며, 자동으로 날짜 형식으로 변환됩니다. 이때 `->toDateString()`를 해줄 필요가 없습니다.

```php
// 이럴땐
$todayUsers = User::whereDate('created_at', now()->toDateString())->get();
// 전환할 필요가 없습니다. 그냥 now()를 쓰세요
$todayUsers = User::whereDate('created_at', now())->get();
```

### 첫 글자로 그룹화 <a id="grouping-by-first-letter"></a>

사용자 지정 조건별로 Eloquent 결과를 그룹화 할 수 있습니다. 사용자 이름의 첫 글자로 그룹화하는 방법은 다음과 같습니다.

```php
$users = User::all()->groupBy(function($item) {
    return $item->name[0];
});
```

### 열 업데이트 안 함 <a id="never-update-the-column"></a>

한 번만 설정하고 다시 업데이트하지 않으려는 DB 열이있는 경우 뮤 테이터를 사용하여 Eloquent Model에 해당 제한을 설정할 수 있습니다.

```php
class User extends Model
{
    public function setEmailAttribute($value)
    {
        if ($this->email) {
            return;
        }

        $this->attributes['email'] = $value;
    }
}
```

### 여러개를 찾기 <a id="find-many"></a>

Eloquent 메소드 `find()` 는 여러 매개 변수를 받아 들일 수 있으며, 하나의 모델이 아닌 발견 된 모든 레코드의 컬렉션을 반환합니다.

```php
// Eloquent Model을 반환합니다
$user = User::find(1);
// Eloquent Collection을 반환합니다
$users = User::find([1,2,3]);
```

### Key로 찾기 <a id="find-by-key"></a>

정확히 어떤 필드가 기본 키인지에 따라 자동으로 처리하는 `whereKey()` 메서드를 사용하여 여러 레코드를 찾을 수도 있습니다 ( 기본값은 `id` 지만 Eloquent 모델에서 재정의 할 수 있습니다).

```php
$users = User::whereKey([1,2,3])->get();
```

### 자동 증가 대신 UUID 사용 <a id="use-uuid-instead-of-auto-increment"></a>

모델에서 자동 증분 ID를 사용하고 싶지 않습니까?

마이그레이션:

```php
Schema::create('users', function (Blueprint $table) {
    // $table->increments('id');
    $table->uuid('id')->unique();
});
```

모델:

```php
class User extends Model
{
    public $incrementing = false;
    protected $keyType = 'string';

    protected static function boot()
    {
        parent::boot();

        User::creating(function ($model) {
            $model->setId();
        });
    }

    public function setId()
    {
        $this->attributes['id'] = Str::uuid();
    }
}
```

### Laravel에서 sub select 사용법<a id="sub-selects-in-laravel-way"></a>

Laravel 6에서는 Eloquent 문에서 addSelect()를 사용하여 추가 된 열에 대해 계산을 할 수 있습니다.

```php
return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderBy('arrived_at', 'desc')
    ->limit(1)
])->get();
```

### 일부 열 숨기기 <a id="hide-some-columns"></a>

Eloquent 쿼리를 수행 할 때 특정 필드가 반환되는 것을 숨기려면 가장 빠른 방법 중 하나는 Collection 결과에 `->makeHidden()` 을 추가하는 것입니다.

```php
$users = User::all()->makeHidden(['email_verified_at', 'deleted_at']);
```

### 정확한 DB 오류 <a id="exact-db-error"></a>

Eloquent Query 예외를 잡으려면 기본 Exception 클래스 대신 특정 `QueryException`을 사용하면 오류의 정확한 SQL 코드를 얻을 수 있습니다.

```php
try {
    // Some Eloquent/SQL statement
} catch (\Illuminate\Database\QueryException $e) {
    if ($e->getCode() === '23000') { // integrity constraint violation
        return back()->withError('Invalid data');
    }
}
```

### 쿼리 빌더에서의 소프트 삭제 <a id="soft-deletes-with-query-builder"></a>

소프트 삭제는 Eloquent를 사용할 때는 삭제된 항목을 제외하지만 Query Builder를 사용하면 작동하지 않는다는 것을 잊지 마십시오.

```php
// Will exclude soft-deleted entries
$users = User::all();

// Will NOT exclude soft-deleted entries
$users = DB::table('users')->get();
```

### 올바른 오래된 SQL 쿼리 <a id="good-old-sql-query"></a>

DB 스키마에서 무언가를 변경하는 것과 같은 결과를 얻지 않고 간단한 SQL 쿼리를 실행해야하는 경우 `DB::statement()` 됩니다.

```
DB::statement('DROP TABLE users');
DB::statement('ALTER TABLE projects AUTO_INCREMENT=123');
```

### DB 트랜잭션 사용 <a id="use-db-transactions></a>

두 개의 DB 작업을 수행하고 두 번째 작업에서 오류가 발생할 수있는 경우 첫 번째 작업을 롤백해야합니다.

이를 위해 DB 트랜잭션을 사용하는 것이 좋습니다. Laravel에서는 정말 쉽습니다.

```php
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});
```

### 업데이트 또는 생성 <a id="update-or-create"></a>

레코드가 있는지 확인한 다음 업데이트하거나 새 레코드를 만들어야하는 경우 한 문장으로 수행 할 수 있습니다. Eloquent 메서드 `updateOrCreate()`를 사용합니다.

```php
// Instead of this
$flight = Flight::where('departure', 'Oakland')
    ->where('destination', 'San Diego')
    ->first();
if ($flight) {
    $flight->update(['price' => 99, 'discounted' => 1]);
} else {
	$flight = Flight::create([
	    'departure' => 'Oakland',
	    'destination' => 'San Diego',
	    'price' => 99,
	    'discounted' => 1
	]);
}
// Do it in ONE sentence
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

### 저장시 캐시 삭제 <a id="forget-cache-on-save"></a>

[@pratiksh404](https://github.com/pratiksh404) 가 제공 한 팁

컬렉션을 제공하는 `posts` 과 같은 캐시 키가 있고 새 저장소 또는 업데이트에서 해당 캐시 키를 잊고 싶다면 모델에서 정적 `saving` 기능을 호출 할 수 있습니다.

```php
class Post extends Model
{
    // Forget cache key on storing or updating
    public static function boot()
    {
        parent::boot();
        static::saving(function () {
           Cache::forget('posts');
        });
    }
}
```

### Created_at 및 Updated_at의 형식 변경 <a id="change-format-of-created_at-and-updated_at">

[@syofyanzuhad](https://github.com/syofyanzuhad) 가 제공 한 팁

`created_at` 형식을 변경하려면 다음과 같이 모델에 메소드를 추가 할 수 있습니다.

```
public function getCreatedAtFormattedAttribute()
{
   return $this->created_at->format('H:i d, M Y');
}
```

따라서 필요할 때 `$entry->created_at_formatted` 사용할 수 있습니다. 다음과 같이 `created_at` 속성이 반환됩니다 : `04:19 23, Aug 2020` .

또한 `updated_at` 속성의 형식을 변경하려면 다음 메소드를 추가 할 수 있습니다.

```
public function getUpdatedAtFormattedAttribute()
{
   return $this->updated_at->format('H:i d, M Y');
}
```

따라서 필요할 때 `$entry->updated_at_formatted` 사용할 수 있습니다. 다음과 같이 `updated_at` 속성이 반환됩니다 : `04:19 23, Aug 2020` .

### JSON에 배열 유형 저장 <a id="storing-array-type-into-json"></a>

[@pratiksh404](https://github.com/pratiksh404) 가 제공 한 팁

배열을 사용하는 입력 필드가 있고이를 JSON으로 저장해야하는 경우 모델에서 `$casts` 속성을 사용할 수 있습니다. 여기 `images` 는 JSON 속성입니다.

```
protected $casts = [
    'images' => 'array',
];
```

따라서 JSON으로 저장할 수 있지만 DB에서 검색하면 배열로 사용할 수 있습니다.

### 모델 사본 만들기 <a id="make-a-copy-of-the-model"></a>

두 개의 매우 유사한 모델 (배송 주소 및 청구 주소)이 있고 서로 사본을 만들어야하는 경우 `replicate()` 메서드를 사용하고 그 후에 일부 속성을 변경할 수 있습니다.

[공식 문서의](https://laravel.com/docs/8.x/eloquent#replicating-models) 예 :

```php
$shipping = Address::create([
    'type' => 'shipping',
    'line_1' => '123 Example Street',
    'city' => 'Victorville',
    'state' => 'CA',
    'postcode' => '90001',
]);

$billing = $shipping->replicate()->fill([
    'type' => 'billing'
]);

$billing->save();
```

## 모델 관계 <a id="models-relations"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (DB 모델 및 Eloquent)](#db-models-and-eloquent) ➡️ [다음 (마이그레이션)](#migrations)

- [Eloquent 관계에 대한 OrderBy](#orderby-on-eloquent-relationships)
- [조건부 관계](#conditional-relationships)
- [Raw DB 쿼리: havingRaw()](#raw-db-queries-havingraw)
- [Eloquent has()를 더 깊게](#eloquent-has-deeper)
- [Has Many. 정확히 얼마나 많은 것을 원하나요?](#has-many-how-many-exactly)
- [기본 모델](#default-model)
- [hasMany를 이용한 다중 생성](#use-hasmany-to-create-many)
- [빠른 로딩에서 필요한 컬럼만 사용하기](#eager-loading-with-exact-columns)
- [쉽게 상위 모델의 updated_at 갱신하기](#touch-parent-updated_at-easily)
- [관계가 존재하는지 항상 확인하세요](#always-check-if-relationship-exists)
- [withCount()를 사용하여 하위 관계 레코드의 갯수 확인](#use-withcount-to-calculate-child-relationships-records)
- [관계에서 필터 쿼리 추가하기](#extra-filter-query-on-relationships)
- [관계를 항상 로드하지만 동적으로도 로드하기](#load-relationships-always-but-dynamically)
- [belongsTo 대신 hasMany를 사용하세요.](#instead-of-belongsto-use-hasmany)
- [피벗 테이블 이름 바꾸기](#rename-pivot-table)
- [한 줄로 상위 모델 업데이트](#update-parent-in-one-line)
- [Laravel 7이상에서 외래 키](#laravel-7-foreign-keys)
- [두 "whereHas"결합](#combine-two-wherehas)
- [관계 메서드가 있는지 확인하기](#check-if-relationship-method-exists)
- [추가 관계가 있는 피벗 테이블](#pivot-table-with-extra-relations)
- [즉석으로 관계 갯수 구해오기](#load-count-on-the-fly)
- [관계 순서 무작위 화](#randomize-relationship-order)

### Eloquent 관계에 대한 OrderBy <a id="orderby-on-eloquent-relationships"></a>

Eloquent 관계에서 직접 orderBy()를 지정할 수 있습니다.

```php
public function products()
{
    return $this->hasMany(Product::class);
}

public function productsByName()
{
    return $this->hasMany(Product::class)->orderBy('name');
}
```

### 조건부 관계 <a id="conditional-relationships"></a>

추가 "where"조건과 함께 동일한 관계를 자주 사용하는 경우 별도의 관계 방법을 만들 수 있습니다.

모델:

```php
public function comments()
{
    return $this->hasMany(Comment::class);
}

public function approved_comments()
{
    return $this->hasMany(Comment::class)->where('approved', 1);
}
```

### Raw DB 쿼리: havingRaw() <a id="raw-db-queries-havingraw"></a>

`havingRaw()` 이후의 `groupBy()` 함수를 포함하여 다양한 위치에서 RAW DB 쿼리를 사용할 수 있습니다.

```php
Product::groupBy('category_id')->havingRaw('COUNT(*) > 1')->get();
```

### Eloquent has()를 더 깊게 <a id="eloquent-has-deeper"></a>

Eloquent `has()` 함수를 사용하여 두 단계의 관계를 쿼리 할 수 있습니다!

```php
// Author -> hasMany(Book::class);
// Book -> hasMany(Rating::class);
$authors = Author::has('books.ratings')->get();
```

### Has Many. 정확히 얼마나 많은 것을 원하나요? <a id="has-many-how-many-exactly"></a>

Eloquent `hasMany()` 관계에서 지정한 양의 하위 레코드가 있는 레코드를 필터링 할 수 있습니다.

```php
// Author -> hasMany(Book::class)
$authors = Author::has('books', '>', 5)->get();
```

### 기본 모델 <a id="default-model"></a>

$post-&gt;user가 존재하지 않는 경우 `{{ $post->user->name }}` 과 같이 호출 할 때 치명적인 오류를 방지하기 위해 `belongsTo` 관계에 기본 모델을 할당 할 수 있습니다.

```php
public function user()
{
    return $this->belongsTo('App\User')->withDefault();
}
```

### hasMany를 이용한 다중 생성 <a id="use-hasmany-to-create-many"></a>

`hasMany()` 관계가있는 경우 `saveMany()` 를 사용하여 "부모"개체의 여러 "자식"항목을 모두 한 문장으로 저장할 수 있습니다.

```php
$post = Post::find(1);
$post->comments()->saveMany([
    new Comment(['message' => 'First comment']),
    new Comment(['message' => 'Second comment']),
]);
```

### 빠른 로딩에서 필요한 컬럼만 사용하기 <a id="eager-loading-with-exact-columns"></a>

Laravel Eager Loading을 실행할 때 관계에서 필요한 컬럼만 가져올 수 있습니다.

```php
$users = App\Book::with('author:id,name')->get();
```

한단계를 거치는 관계에서도 사용 할 수 있습니다.

```php
$users = App\Book::with('author.country:id,name')->get();
```

### 쉽게 상위 모델의 updated_at 갱신하기 <a id="touch-parent-updated_at-easily"></a>

레코드를 업데이트하고 상위 관계의 `updated_at` 열을 갱신하려면 (예 : 새 댓글을 추가하고 게시글 `posts.updated_at`를 갱신하려는 경우) 자식 모델에 `$touches = ['post'];`속성을 추가.

```php
class Comment extends Model
{
    protected $touches = ['post'];
}
```

### 관계가 존재하는지 항상 확인하세요 <a id="always-check-if-relationship-exists"></a>

**지금까지는** `$model->relationship->field`처럼 관계 객체가 여전히 존재하는지 확인하지 않고 사용했습니다.

현재 동작하는 코드 외부, 즉 다른 사람의 대기열 작업 등 어떤 이유로든 관계는 삭제 될 수 있습니다. `if-else` , 또는 `{{ $model->relationship->field ?? '' }}` 또는 (블레이드)`{{ optional($model->relationship)->field }}`를 사용하세요.

### withCount()를 사용하여 하위 관계 레코드의 갯수 확인 <a id="use-withcount-to-calculate-child-relationships-records"></a>

`hasMany()` 관계에서 "자식"항목을 갯수를 확인하려면 쿼리를 따로 작성하지 마세요. 예를 들어, User 모델에 대한 게시물과 댓글이 있을 경우 `withCount()` 사용하세요.

```php
public function index()
{
    $users = User::withCount(['posts', 'comments'])->get();
    return view('users', compact('users'));
}
```

그런 다음 블레이드 파일에서 `{relationship}_count` 속성을 사용하여 해당 갯수를 가져올 수 있습니다.

```blade
@foreach ($users as $user)
<tr>
    <td>{{ $user->name }}</td>
    <td class="text-center">{{ $user->posts_count }}</td>
    <td class="text-center">{{ $user->comments_count }}</td>
</tr>
@endforeach
```

해당 필드로 정렬할 수도 있습니다.

```php
User::withCount('comments')->orderBy('comments_count', 'desc')->get();
```

### 관계에서 필터 쿼리 추가하기 <a id="extra-filter-query-on-relationships"></a>

관계 데이터를 불러올때 클로저 함수에서 몇 가지 제한 또는 순서를 지정할 수 있습니다. 예를 들어, 가장 큰 도시가 3 개만 있는 국가를 얻으려면 다음과 같이 코드를 작성하면 됩니다.

```php
$countries = Country::with(['cities' => function($query) {
    $query->orderBy('population', 'desc');
    $query->take(3);
}])->get();
```

### 관계를 항상 로드하지만 동적으로도 로드하기 <a id="load-relationships-always-but-dynamically"></a>

모델과 함께 항상 로드 할 관계를 지정할 수 있을 뿐만 아니라 생성자 메서드에서 동적으로 처리할 수도 있습니다.

```php
class ProductTag extends Model
{
    protected $with = ['product'];

    public function __construct() {
        parent::__construct();
        $this->with = ['product'];
        
        if (auth()->check()) {
            $this->with[] = 'user';
        }
    }
}
```

### belongsTo 대신 hasMany를 사용하세요. <a id="instead-of-belongsto-use-hasmany"></a>

`belongsTo` 관계에서 하위 레코드를 만들 때 부모의 ID를 전달하는 방식 대신, `hasMany` 관계를 사용하여 더 짧은 문장을 만들 수 있습니다.

```php
// 만약 Post -> belongsTo(User)이고, User -> hasMany(Post) 라면...
// user_id를 넘겨주는 대신...
Post::create([
    'user_id' => auth()->id(),
    'title' => request()->input('title'),
    'post_text' => request()->input('post_text'),
]);

// 이렇게 사용하세요
auth()->user()->posts()->create([
    'title' => request()->input('title'),
    'post_text' => request()->input('post_text'),
]);
```

### 피벗 테이블 이름 바꾸기 <a id="rename-pivot-table"></a>

"pivot"단어의 이름을 바꾸고 관계를 다른 이름으로 부르려면 관계에서 `->as('name')` 를 사용하면됩니다.

모델:

```php
public function podcasts() {
    return $this->belongsToMany('App\Podcast')
        ->as('subscription')
        ->withTimestamps();
}
```

컨트롤러:

```php
$podcasts = $user->podcasts();
foreach ($podcasts as $podcast) {
    // $podcast->pivot->created_at 대신에...
    echo $podcast->subscription->created_at;
}
```

### 한 줄로 상위 모델 업데이트 <a id="update-parent-in-one-line"></a>

`belongsTo()` 관계일 경우 한 문장으로 Eloquent 관계 데이터를 업데이트 할 수 있습니다.

```php
// 만약 Project -> belongsTo(User::class)이라면
$project->user->update(['email' => 'some@gmail.com']);
```

### Laravel 7이상에서 외래 키 <a id="laravel-7-foreign-keys"></a>

Laravel 7부터 마이그레이션시 관계 필드에 대해 두 줄을 작성할 필요가 없습니다. 하나는 필드 용이고 다른 하나는 외래 키용입니다. `foreignId()` 메소드를 사용하십시오.

```php
// Laravel 7 이전
Schema::table('posts', function (Blueprint $table)) {
    $table->unsignedBigInteger('user_id');
    $table->foreign('user_id')->references('id')->on('users');
}

// Laravel 7 부터
Schema::table('posts', function (Blueprint $table)) {
    $table->foreignId('user_id')->constrained();
}

// 또는, 필드가 참조 테이블과 다른 경우
Schema::table('posts', function (Blueprint $table)) {
    $table->foreignId('created_by_id')->constrained('users', 'column');
}
```

### 두 "whereHas"결합 <a id="combine-two-wherehas"></a>

Eloquent에서는 `whereHas()`과 `orDoesntHave()`을 한 문장으로 결합 할 수 있습니다.

```php
User::whereHas('roles', function($query) {
    $query->where('id', 1);
})
->orDoesntHave('roles')
->get();
```

### 관계 메서드가 있는지 확인하기 <a id="check-if-relationship-method-exists"></a>

Eloquent 관계 이름이 동적이고 해당 이름과의 관계가 객체에 존재하는지 확인해야 하는 경우 PHP 함수 `method_exists($object, $methodName)`를 사용하세요.

```php
$user = User::first();
if (method_exists($user, 'roles')) {
	// $user->roles()-> 와 함께 관계 메서드 사용하기...
}
```

### 추가 관계가 있는 피벗 테이블 <a id="pivot-table-with-extra-relations"></a>

다대다 관계에서 피벗 테이블에는 추가 필드 및 다른 모델에 대한 추가 관계가 포함될 수 있습니다.

다음과 같이 별도의 피벗 모델을 생성합니다.

```
php artisan make:model RoleUser --pivot
```

그다음 `->using()` 메서드를 사용하여 `belongsToMany()`에 지정합니다. 그러면 다음 예제와 같이 쩌는 것을 할 수 있습니다.

```php
// app/Models/User.php 에서
public function roles()
{
	return $this->belongsToMany(Role::class)
	    ->using(RoleUser::class)
	    ->withPivot(['team_id']);
}

// app/Models/RoleUser.php 모델은 Model이 아닌 Illuminate\Database\Eloquent\Relations\Pivot을 확장하여 사용합니다;

class RoleUser extends Pivot
{
	public function team()
	{
	    return $this->belongsTo(Team::class);
	}
}

// 그러면 Controller에서 아래와 같이 실행할 수 있습니다.
$firstTeam = auth()->user()->roles()->first()->pivot->team->name;
```

### 즉석으로 관계 갯수 구해오기 <a id="load-count-on-the-fly"></a>

관련 레코드의 갯수를 가져오기 위해 Eloquent의 `withCount()` 메서드 외에도 `loadCount()`를 사용하여 즉석으로 갯수를 가져올 수 도 있습니다.

```php
// 만약 Book이 여러개의 Reviews를 가지고 있다면...
$book = App\Book::first();

$book->loadCount('reviews');
// $book->reviews_count; 를 사용할 수 있습니다

// 또 추가 조건과 함께도 가능합니다
$book->loadCount(['reviews' => function ($query) {
    $query->where('rating', 5);
}]);
```

### 관계 순서 무작위 화 <a id="randomize-relationship-order"></a>

`inRandomOrder()`를 사용하여 Eloquent 쿼리 결과를 무작위화 할 수 있지만 이를 사용하여 쿼리로 로드하는 **relationship** 항목을 무작위화 할 수도 있습니다.

```php
// 퀴즈가 있고 질문을 무작위로 선택하려는 경우...

// 1. 무작위 순서로 질문을 받고 싶다면
$questions = Question::inRandomOrder()->get();

// 2. 무작위 순서로 질문 옵션을 가져오려면 다음과 같이 사용하세요.
$questions = Question::with(['answers' => function($q) {
    $q->inRandomOrder();
}])->inRandomOrder()->get();
```

## 마이그레이션 <a id="migrations"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (모델 관계)](#models-relations) ➡️ [다음 (보기)](#views)

- [부호없는 정수](#unsigned-integer)
- [마이그레이션 순서](#order-of-migrations)
- [시간대가있는 마이그레이션 필드](#migration-fields-with-timezones)
- [데이터베이스 마이그레이션 열 유형](#database-migrations-column-types)
- [기본 타임 스탬프](#default-timestamp)
- [마이그레이션 상태](#migration-status)
- [공백과 함께 마이그레이션 만들기](#create-migration-with-spaces)
- [다른 열 뒤에 열 만들기](#create-column-after-another-column)

### 부호없는 정수 <a id="unsigned-integer"></a>

외래 키 마이그레이션의 경우 `integer()` 대신 `unsignedInteger()` 유형 또는 `integer()->unsigned()`를 사용하세요 . 그렇지 않으면 SQL 오류가 발생할 수 있습니다.

```php
Schema::create('employees', function (Blueprint $table) {
    $table->unsignedInteger('company_id');
    $table->foreign('company_id')->references('id')->on('companies');
    // ...
});
```

다른 열이 `bigInteger()` 유형 인 경우 `unsignedBigInteger()` 사용할 수도 있습니다.

```php
Schema::create('employees', function (Blueprint $table) {
    $table->unsignedBigInteger('company_id');
});
```

### 마이그레이션 순서 <a id="order-of-migrations"></a>

당신은 DB 마이그레이션의 순서를 변경하려면, 단지 다음과 같이 파일에서 타임 스탬프만 바꾸면 됩니다. `2018_08_04_070443_create_posts_table.php` 에서 `2018_07_04_070443_create_posts_table.php` 로 변경. (`2018_08_04` 에서 `2018_07_04` 로 변경).

마이그레이션은 알파벳 순서로 실행됩니다.

### 시간대가 있는 마이그레이션 필드 <a id="migration-fields-with-timezones"></a>

마이그레이션에서 timezone에 대한 timetamps `timestamps()` 뿐만 아니라 `timestampsTz()` 도 있다는 것을 알고 계셨습니까?

```php
Schema::create('employees', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->string('email');
    $table->timestampsTz();
});
```

또한 `dateTimeTz()` , `timeTz()` , `timestampTz()` , `softDeletesTz()` 도 있습니다.

### 데이터베이스 마이그레이션 열 유형 <a id="database-migrations-column-types"></a>

마이그레이션을 위한 재밌는 열 유형이 있습니다. 몇 가지 예를 보여드리겠습니다.

```php
$table->geometry('positions');
$table->ipAddress('visitor');
$table->macAddress('device');
$table->point('position');
$table->uuid('id');
```

[공식 문서](https://laravel.com/docs/master/migrations#creating-columns) 에서 모든 열 유형을 참조하십시오.

### 기본 타임 스탬프 <a id="default-timestamp"></a>

마이그레이션을 생성하는 동안 `useCurrent()` 옵션과 함께 `timestamp()` 열 유형을 사용할 수 있으며 `CURRENT_TIMESTAMP` 를 기본값으로 설정합니다.

```php
$table->timestamp('created_at')->useCurrent();
$table->timestamp('updated_at')->useCurrent();
```

### 마이그레이션 상태 <a id="migration-status"></a>

어떤 마이그레이션이 아직 실행되지 않았는지 확인하고 싶을때, 굳이 데이터베이스에서 "migrations"테이블을 볼 필요가 없습니다. `php artisan migrate:status` 명령을 실행해보세요.

결과 예 :

```
+------+------------------------------------------------+-------+
| Ran? | Migration                                      | Batch |
+------+------------------------------------------------+-------+
| Yes  | 2014_10_12_000000_create_users_table           | 1     |
| Yes  | 2014_10_12_100000_create_password_resets_table | 1     |
| No   | 2019_08_19_000000_create_failed_jobs_table     |       |
+------+------------------------------------------------+-------+
```

### 공백과 함께 마이그레이션 만들기 <a id="create-migration-with-spaces"></a>

`make:migration` 명령어를 입력 할 때 `create_transactions_table`과 같이 부분 사이에 반드시 밑줄 `_` 기호를 사용할 필요는 없습니다. 이름을 따옴표로 묶은 다음 밑줄 대신 공백을 사용할 수 있습니다.

```php
// This works
php artisan make:migration create_transactions_table

// But this works too
php artisan make:migration "create transactions table"
```

출처 : [Twitter의 Steve O](https://twitter.com/stephenoldham/status/1353647972991578120)

### 다른 열 뒤에 열 만들기 <a id="create-column-after-another-column"></a>

기존 테이블에 새 열을 추가할 때 반드시 마지막 열로 만들 필요는 없습니다. 생성해야 하는 열을 특정 열의 뒤로 지정할 수 있습니다.

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->after('email');
});
```

## 뷰 <a id="views"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (마이그레이션)](#migrations) ➡️ [다음 (라우팅)](#routing)

- [foreach의 $loop 변수](#loop-variable-in-foreach)
- [뷰 파일이 있습니까?](#does-view-file-exist)
- [오류 코드 블레이드 페이지](#error-code-blade-pages)
- [컨트롤러 없이 뷰 사용하기](#view-without-controllers)
- [블레이드 @auth](#blade-auth)
- [블레이드의 2 단계 $loop 변수](#two-level-loop-variable-in-blade)
- [나만의 블레이드 지시문 만들기](#create-your-own-blade-directive)
- [블레이드 지시문 : IncludeIf, IncludeWhen, IncludeFirst](#blade-directives-includeif-includewhen-includefirst)

### foreach의 $loop 변수 <a id="loop-variable-in-foreach"></a>

foreach 루프 내에서 `$loop` 변수를 사용하여 현재 항목이 처음 / 마지막인지 확인합니다.

```blade
@foreach ($users as $user)
     @if ($loop->first)
        This is the first iteration.
     @endif

     @if ($loop->last)
        This is the last iteration.
     @endif

     <p>This is user {{ $user->id }}</p>
@endforeach
```

`$loop->iteration` 또는 `$loop->count` 와 같은 다른 속성도 있습니다. [공식 문서](https://laravel.com/docs/master/blade#the-loop-variable) 에서 자세히 알아보십시오.

### 뷰 파일이 있습니까? <a id="does-view-file-exist"></a>

실제로 뷰를 로드하기 전에 해당 뷰 파일이 있는지 확인할 수 있습니다.

```php
if (view()->exists('custom.page')) {
 // Load the view
}
```

로드 할 뷰 배열을 사용 할 수도 있으며, 이때는 처음 존재하는 뷰만 실제로 로드됩니다.

```php
return view()->first(['custom.dashboard', 'dashboard'], $data);
```

### 오류 코드 블레이드 페이지 <a id="error-code-blade-pages"></a>

500과 같은 일부 HTTP 코드에 대한 특정 오류 페이지를 생성하려면 이 코드를 파일 이름으로 `resources/views/errors/500.blade.php` 또는 `403.blade.php` 등으로 블레이드 파일을 만드세요. 해당 오류 코드에 맞게 자동으로 로드됩니다.

### 컨트롤러 없이 뷰 사용하기 <a id="view-without-controllers"></a>

라우트가 특정 뷰만 보여준다면 Controller를 생성하지 말고 `Route::view()` 함수를 사용하십시오.

```php
// 이것처럼 되어있거나
Route::get('about', 'TextsController@about');
// 이것처럼 되어있다면
class TextsController extends Controller
{
    public function about()
    {
        return view('texts.about');
    }
}
// 이렇게 사용하세요
Route::view('about', 'texts.about');
```

### 블레이드 @auth <a id="blade-auth"></a>

로그인 한 사용자를 확인하기위해 if 문 대신 `@auth` 지시문을 사용하세요.

일반적인 방법 :

```blade
@if(auth()->user())
    // The user is authenticated.
@endif
```

짧게 :

```blade
@auth
    // The user is authenticated.
@endauth
```

반대는 `@guest` 지시문입니다.

```blade
@guest
    // The user is not authenticated.
@endguest
```

### 블레이드의 2 단계 $loop 변수 <a id="two-level-loop-variable-in-blade"></a>

Blade의 foreach에서는 2 단계 루프에서도 $loop 변수를 사용하여 상위 변수에 접근 할 수 있습니다.

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

### 나만의 블레이드 지시문 만들기 <a id="create-your-own-blade-directive"></a>

진짜 쉽습니다. `app/Providers/AppServiceProvider.php` 에 자신만의 메서드를 추가하기만 하면됩니다. 예를 들어, `<br>` 태그를 엔터로 바꾸려면 다음과 같이하십시오.

```blade
// 블레이드에서
<textarea>@br2nl($post->post_text)</textarea>
```

이 지시문을 AppServiceProvider의 `boot()` 메서드에 추가합니다.

```php
public function boot()
{
    Blade::directive('br2nl', function ($string) {
        return "<?php echo preg_replace('/\<br(\s*)?\/?\>/i', \"\n\", $string); ?>";
    });
}
```

### 블레이드 지시문 : IncludeIf, IncludeWhen, IncludeFirst <a id="blade-directives-includeif-includewhen-includefirst"></a>

Blade 부분적으로 사용하는 파일이 실제로 존재하는지 확실하지 않은 경우 다음 조건 명령을 사용할 수 있습니다.

블레이드 파일이 있는 경우에만 헤더를 로드합니다.

```blade
@includeIf('partials.header')
```

role_id가 1 인 사용자에 대해서만 헤더를로드합니다.

```blade
@includeWhen(auth()->user()->role_id == 1, 'partials.header')
```

adminlte.header를 로드하려고 시도합니다. 해당 파일이 없을 경우에만 default.header를 로드합니다.

```blade
@includeFirst('adminlte.header', 'default.header')
```

## 라우팅 <a id="routing"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (뷰)](#views) ➡️ [다음 (유효성 검사)](#validation)

- [그룹 내의 라우팅 그룹](#route-group-within-a-group)
- [와일드 카드 하위 도메인](#wildcard-subdomains)
- [라우트 뒤에는 무엇이 있을까요?](#whats-behind-the-routes)
- [라우트에서 모델 바인딩 : 키를 지정 할 수 있습니다.](#route-model-binding-you-can-define-a-key)
- [Routes 파일에서 컨트롤러로 빠르게 이동](#quickly-navigate-from-routes-file-to-controller)
- [대체 라우트 : 일치하는 다른 경로가 없는 경우](#route-fallback-when-no-other-route-is-matched)
- [정규식을 사용한 라우트 매개 변수 유효성 검사](#route-parameters-validation-with-regexp)
- [속도 제한 : 글로벌 및 게스트 / 사용자 용](#rate-limiting-global-and-for-guestsusers)
- [라우트에 매개 변수로 쿼리스트링을 추가하기](#query-string-parameters-to-routes)
- [파일별로 라우트 분리](#separate-routes-by-files)
- [리소스 동사 번역](#translate-resource-verbs)
- [커스텀 리소스 라우트 네임](#custom-resource-route-names)
- [더 읽기 쉬운 라우트 목록](#more-readable-route-list)

### 그룹 내의 라우팅 그룹 <a id="route-group-within-a-group"></a>

라우트에서 그룹 내 그룹을 생성하여, 특정 미들웨어를 "상위"그룹의 일부 URL에만 할당 할 수 있습니다.

```php
Route::group(['prefix' => 'account', 'as' => 'account.'], function() {
    Route::get('login', 'AccountController@login');
    Route::get('register', 'AccountController@register');
    
    Route::group(['middleware' => 'auth'], function() {
        Route::get('edit', 'AccountController@edit');
    });
});
```

### 와일드 카드 하위 도메인 <a id="wildcard-subdomains"></a>

동적인 하위 도메인 이름으로 라우트 그룹을 생성하고 해당 값을 모든 경로에 전달할 수 있습니다.

```php
Route::domain('{username}.workspace.com')->group(function () {
    Route::get('user/{id}', function ($username, $id) {
        //
    });
});
```

### 라우트 뒤에는 무엇이 있을까요? <a id="whats-behind-the-routes"></a>

`Auth::routes()` 에 실제로 어떤 경로들이 있는지 알고 싶나요? Laravel 7부터는 별도의 패키지에 있는 `/vendor/laravel/ui/src/AuthRouteMethods.php` 파일을 확인해보세요.

```php
public function auth()
{
    return function ($options = []) {
        // Authentication Routes...
        $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
        $this->post('login', 'Auth\LoginController@login');
        $this->post('logout', 'Auth\LoginController@logout')->name('logout');
        // Registration Routes...
        if ($options['register'] ?? true) {
            $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
            $this->post('register', 'Auth\RegisterController@register');
        }
        // Password Reset Routes...
        if ($options['reset'] ?? true) {
            $this->resetPassword();
        }
        // Password Confirmation Routes...
        if ($options['confirm'] ?? class_exists($this->prependGroupNamespace('Auth\ConfirmPasswordController'))) {
            $this->confirmPassword();
        }
        // Email Verification Routes...
        if ($options['verify'] ?? false) {
            $this->emailVerification();
        }
    };
}
```

Laravel 7 이전이라면 `/vendor/laravel/framework/src/illuminate/Routing/Router.php` 파일을 확인해보세요.

### 라우트에서 모델 바인딩 : 키를 지정 할 수 있습니다. <a id="route-model-binding-you-can-define-a-key"></a>

라우트에 모델을 `Route::get('api/users/{user}', function (App\User $user) { … }` 이런식으로 ID 필드를 기준으로 바인딩 할 수 있을 뿐만 아니라, `{user}`를 `username` 필드로 변경하고 싶다면 다음과 같이 모델에 입력하면 됩니다.

```php
public function getRouteKeyName() {
    return 'username';
}
```

### Routes 파일에서 컨트롤러로 빠르게 이동 <a id="quickly-navigate-from-routes-file-to-controller"></a>

이것은 Laravel 8 이전에는 선택 사항이었으며 Laravel 8에서 라우팅의 표준 기본 구문이되었습니다.

다음과 같이 라우팅하는 대신

```php
Route::get('page', 'PageController@action');
```

컨트롤러를 클래스로 지정할 수 있습니다.

```php
Route::get('page', [\App\Http\Controllers\PageController::class, 'action']);
```

그런 다음 PhpStorm에서 **PageController** 를 클릭하고 수동으로 검색하는 대신 Controller로 직접 이동할 수 있습니다.

또 길이를 줄이려면 다음과 같이 Routes 파일의 맨 위에 use 구분을 추가하십시오.

```php
use App\Http\Controllers\PageController;

// 요롷게~
Route::get('page', [PageController::class, 'action']);
```

### 대체 라우트 : 일치하는 다른 경로가 없는 경우 <a id="route-fallback-when-no-other-route-is-matched"></a>

찾을 수 없는 경로에 대한 추가 로직을 지정하려면 기본 404 페이지를 표시하는 대신 라우트 파일의 맨 끝에 특수 경로를 만들 수 있습니다.

```php
Route::group(['middleware' => ['auth'], 'prefix' => 'admin', 'as' => 'admin.'], function () {
    Route::get('/home', 'HomeController@index');
    Route::resource('tasks', 'Admin\TasksController');
});

// Some more routes....
Route::fallback(function() {
    return 'Hm, why did you land here somehow?';
});
```

### 정규식을 사용한 라우트 매개 변수 유효성 검사 <a id="route-parameters-validation-with-regexp"></a>

"where"매개 변수를 사용하여 경로에서 직접 매개 변수를 검증 할 수 있습니다. 일반적으로 경로에 `fr/blog` 및 `en/article/333` 과 같이 언어 로케일로 접두사를 지정하는 경우가 있습니다. 이때 두 개의 첫 글자가 영어가 아닌 다른 언어가 사용되지 않도록하려면 어떻게해야 할까요?

`routes/web.php` :

```php
Route::group([
    'prefix' => '{locale}',
    'where' => ['locale' => '[a-zA-Z]{2}']
], function () {
    Route::get('/', 'HomeController@index');
    Route::get('article/{id}', 'ArticleController@show');
});
```

### 속도 제한 : 글로벌 및 게스트 / 사용자 용 <a id="rate-limiting-global-and-for-guestsusers"></a>

`throttle:60,1`을 사용하여 일부 URL은 분당 최대 60 회 호출되도록 제한 할 수 있습니다.

```php
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

또한 게스트 및 로그인 한 사용자에 대해 별도로 지정 할 수 있습니다.

```php
// 게스트에 대해 최대 10 개의 요청, 인증 된 사용자에 대해 60 개
Route::middleware('throttle:10|60,1')->group(function () {
    //
});
```

또한 DB 필드 users.rate_limit를 사용해 특정 사용자에 대한 제한 지정 할 수 있습니다.

```php
Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

### 라우트에 매개 변수로 쿼리스트링을 추가하기 <a id="query-string-parameters-to-routes"></a>

라우트에 추가 매개 변수를 전달하면 배열의 키/값이 생성 될 URL의 쿼리 문자열에 자동으로 추가됩니다.

```php
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']); // Result: /user/1/profile?photos=yes
```

### 파일별로 라우트 분리 <a id="separate-routes-by-files"></a>

특정 "섹션"에 관한 경로가 셋트로 있는 경우, 별도의 `routes/XXXXX.php` 같은 파일로 분리해서 `routes/web.php`에 포함시킬 수 있습니다

Taylor Otwell이 직접 만든 [Laravel Breeze](https://github.com/laravel/breeze/blob/1.x/stubs/routes/web.php)의 `routes/auth.php` 예제 :

```php
Route::get('/', function () {
    return view('welcome');
});

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth'])->name('dashboard');

require __DIR__.'/auth.php';
```

그런 다음, `routes/auth.php` :

```php
use App\Http\Controllers\Auth\AuthenticatedSessionController;
use App\Http\Controllers\Auth\RegisteredUserController;
// ... more controllers

use Illuminate\Support\Facades\Route;

Route::get('/register', [RegisteredUserController::class, 'create'])
                ->middleware('guest')
                ->name('register');

Route::post('/register', [RegisteredUserController::class, 'store'])
                ->middleware('guest');

// ... A dozen more routes
```

하지만 이 `include()`는 별도의 라우트 파일에 접두사 / 미들웨어에 대해 동일한 설정일 경우에만 사용해야합니다. 그렇지 않으면 `app/Providers/RouteServiceProvider`에서 그룹화하는 것이 좋습니다.

```php
public function boot()
{
    $this->configureRateLimiting();

    $this->routes(function () {
        Route::prefix('api')
            ->middleware('api')
            ->namespace($this->namespace)
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->namespace($this->namespace)
            ->group(base_path('routes/web.php'));

        // ... Your routes file listed next here
    });
}
```

### 리소스 동사 번역 <a id="translate-resource-verbs"></a>

리소스 컨트롤러를 사용하지만 SEO 목적으로 URL 동사를 영어가 아닌 것으로 변경하려는 경우 `/create` 대신 스페인어 `/crear` 를 원하면 `App\Providers\RouteServiceProvider` `Route::resourceVerbs()` 메서드를 사용하여 변경 할 수 있습니다.

```php
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);

    // ...
}
```

### 사용자 지정 리소스 경로 이름 <a id="custom-resource-route-names"></a>

리소스 컨트롤러를 사용할 때 `routes/web.php` 에서 `->names()` 매개 변수를 지정할 수 있으므로 브라우저의 URL 접두사와 Laravel 프로젝트 전체에서 사용하는 경로 이름 접두사가 다를 수 있습니다.

```php
Route::resource('p', ProductController::class)->names('products');
```

따라서 위의 코드는 `/p` , `/p/{id}` , `/p/{id}/edit` 등과 같은 URL을 생성합니다.하지만 코드에서 `route('products.index')` , `route('products.create')` 등

### 더 읽기 쉬운 라우트 목록 <a id="more-readable-route-list"></a>

"php artisan route : list"를 실행 한 다음 목록이 너무 많은 공간을 차지하고 읽기 어렵다는 것을 깨달은 적이 있습니까?

해결책은 다음과 같습니다 : `php artisan route:list --compact`

그러면 6 개 열 대신 3 개 열이 표시됩니다. Method / URI / Action 만 표시됩니다.

```
+----------+---------------------------------+-------------------------------------------------------------------------+
| Method   | URI                             | Action                                                                  |
+----------+---------------------------------+-------------------------------------------------------------------------+
| GET|HEAD | /                               | Closure                                                                 |
| GET|HEAD | api/user                        | Closure                                                                 |
| POST     | confirm-password                | App\Http\Controllers\Auth\ConfirmablePasswordController@store           |
| GET|HEAD | confirm-password                | App\Http\Controllers\Auth\ConfirmablePasswordController@show            |
| GET|HEAD | dashboard                       | Closure                                                                 |
| POST     | email/verification-notification | App\Http\Controllers\Auth\EmailVerificationNotificationController@store |
| POST     | forgot-password                 | App\Http\Controllers\Auth\PasswordResetLinkController@store             |
| GET|HEAD | forgot-password                 | App\Http\Controllers\Auth\PasswordResetLinkController@create            |
| POST     | login                           | App\Http\Controllers\Auth\AuthenticatedSessionController@store          |
| GET|HEAD | login                           | App\Http\Controllers\Auth\AuthenticatedSessionController@create         |
| POST     | logout                          | App\Http\Controllers\Auth\AuthenticatedSessionController@destroy        |
| POST     | register                        | App\Http\Controllers\Auth\RegisteredUserController@store                |
| GET|HEAD | register                        | App\Http\Controllers\Auth\RegisteredUserController@create               |
| POST     | reset-password                  | App\Http\Controllers\Auth\NewPasswordController@store                   |
| GET|HEAD | reset-password/{token}          | App\Http\Controllers\Auth\NewPasswordController@create                  |
| GET|HEAD | verify-email                    | App\Http\Controllers\Auth\EmailVerificationPromptController@__invoke    |
| GET|HEAD | verify-email/{id}/{hash}        | App\Http\Controllers\Auth\VerifyEmailController@__invoke                |
+----------+---------------------------------+-------------------------------------------------------------------------+
```

원하는 정확한 열을 지정할 수도 있습니다.

`php artisan route:list --columns=Method,URI,Name`

```
+----------+---------------------------------+---------------------+
| Method   | URI                             | Name                |
+----------+---------------------------------+---------------------+
| GET|HEAD | /                               |                     |
| GET|HEAD | api/user                        |                     |
| POST     | confirm-password                |                     |
| GET|HEAD | confirm-password                | password.confirm    |
| GET|HEAD | dashboard                       | dashboard           |
| POST     | email/verification-notification | verification.send   |
| POST     | forgot-password                 | password.email      |
| GET|HEAD | forgot-password                 | password.request    |
| POST     | login                           |                     |
| GET|HEAD | login                           | login               |
| POST     | logout                          | logout              |
| POST     | register                        |                     |
| GET|HEAD | register                        | register            |
| POST     | reset-password                  | password.update     |
| GET|HEAD | reset-password/{token}          | password.reset      |
| GET|HEAD | verify-email                    | verification.notice |
| GET|HEAD | verify-email/{id}/{hash}        | verification.verify |
+----------+---------------------------------+---------------------+
```

## 유효성 검사 <a id="validation"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (라우팅)](#routing) ➡️ [다음 (컬렉션)](#collections)

- [이미지 유효성 검사](#image-validation)
- [사용자 지정 유효성 검사 오류 메시지](#custom-validation-error-messages)
- ["지금"또는 "어제"단어로 날짜 확인](#validate-dates-with-now-or-yesterday-words)
- [일부 조건이있는 유효성 검사 규칙](#validation-rule-with-some-conditions)
- [기본 검증 메시지 변경](#change-default-validation-messages)
- [검증 준비](#prepare-for-validation)
- [첫 번째 유효성 검사 오류시 중지](#stop-on-first-validation-error)

### 이미지 유효성 검사 <a id="image-validation"></a>

업로드 된 이미지의 유효성을 검사하는 동안 필요한 크기를 지정할 수 있습니다.

```php
['photo' => 'dimensions:max_width=4096,max_height=4096']
```

### 사용자 지정 유효성 검사 오류 메시지 <a id="custom-validation-error-messages"></a>

**필드** , **규칙** 및 **언어** 별로 유효성 검사 오류 메시지를 사용자 정의 할 수 있습니다. 적절한 배열 구조로 특정 언어 파일 `resources/lang/xx/validation.php` 를 생성하기 만하면됩니다.

```php
'custom' => [
     'email' => [
        'required' => 'We need to know your e-mail address!',
     ],
],
```

### "지금"또는 "어제"단어로 날짜 확인 <a id="validate-dates-with-now-or-yesterday-words"></a>

이전 / 이후 규칙에 따라 날짜의 유효성을 검사하고 `tomorrow` , `now` , `yesterday` 와 같은 다양한 문자열을 매개 변수로 전달할 수 있습니다. 예 : `'start_date' => 'after:now'` . 내부적으로는 strtotime()을 사용하고 있습니다.

```php
$rules = [
    'start_date' => 'after:tomorrow',
    'end_date' => 'after:start_date'
];
```

### 일부 조건이있는 유효성 검사 규칙 <a id="validation-rule-with-some-conditions"></a>

유효성 검사 규칙이 일부 조건에 의존하는 경우 `FormRequest` 클래스에 `withValidator()` 를 추가하여 규칙을 수정하고 여기에서 사용자 지정 논리를 지정할 수 있습니다. 예를 들어 일부 사용자 역할에 대해서만 유효성 검사 규칙을 추가하려는 경우입니다.

```php
use Illuminate\Validation\Validator;
class StoreBlogCategoryRequest extends FormRequest {
    public function withValidator(Validator $validator) {
        if (auth()->user()->is_admin) {
            $validator->addRules(['some_secret_password' => 'required']);
        }
    }
}
```

### 기본 검증 메시지 변경 <a id="change-default-validation-messages"></a>

특정 필드 및 특정 유효성 검사 규칙에 대한 기본 유효성 검사 오류 메시지를 변경하려면 `FormRequest` 클래스에 `messages()` 메서드를 추가하면됩니다.

```php
class StoreUserRequest extends FormRequest
{
    public function rules()
    {
        return ['name' => 'required'];
    }
    
    public function messages()
    {
        return ['name.required' => 'User name should be real name'];
    }
}
```

### 검증 준비 <a id="prepare-for-validation"></a>

기본 Laravel 유효성 검사 전에 일부 필드를 수정하려는 경우, 즉 해당 필드를 "준비"하려면 무엇을 추측해야합니다. 이것을 위한`FormRequest` 클래스에 `prepareForValidation()` 메소드가 있습니다.

```php
protected function prepareForValidation()
{
    $this->merge([
        'slug' => Illuminate\Support\Str::slug($this->slug),
    ]);
}
```

### 첫 번째 유효성 검사 오류시 중지 <a id="stop-on-first-validation-error"></a>

기본적으로 Laravel 유효성 검사 오류는 모든 유효성 검사 규칙을 확인하여 목록으로 반환됩니다. 그러나 첫 번째 오류 후에 프로세스를 중지하려면 `bail` 이라는 유효성 검사 규칙을 사용하십시오.

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

## 컬렉션 <a id="collections"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (검증)](#validation) ➡️ [다음 (인증)](#auth)

- [컬렉션에서 NULL로 필터링하지 마십시오.](#dont-filter-by-null-in-collections)
- [사용자 정의 콜백 함수가있는 콜렉션에서 groupBy 사용](#use-groupby-on-collections-with-custom-callback-function)
- [한 행의 여러 수집 방법](#multiple-collection-methods-in-a-row)
- [페이지네이션 합계 계산](#calculate-sum-with-pagination)

### 컬렉션에서 NULL로 필터링하지 마십시오. <a id="dont-filter-by-null-in-collections"></a>

Eloquent에서 NULL로 필터링 할 수 있지만 **컬렉션을** 추가로 필터링하는 경우 빈 문자열로 필터링하면 해당 필드에 더 이상 "null"이 없습니다.

```php
// This works
$messages = Message::where('read_at is null')->get();

// Won’t work - will return 0 messages
$messages = Message::all();
$unread_messages = $messages->where('read_at is null')->count();

// Will work
$unread_messages = $messages->where('read_at', '')->count();
```

### 사용자 정의 콜백 함수가있는 콜렉션에서 groupBy 사용 <a id="use-groupby-on-collections-with-custom-callback-function"></a>

데이터베이스의 직접 열이 아닌 일부 조건에 따라 결과를 그룹화하려면 클로저 기능을 제공하여 수행 할 수 있습니다.

예를 들어, 등록일별로 사용자를 그룹화하려는 경우 코드는 다음과 같습니다.

```php
$users = User::all()->groupBy(function($item) {
    return $item->created_at->format('Y-m-d');
});
```

⚠️주의 : `Collection` 클래스에서 수행되므로 데이터베이스에서 결과를 가져온 **후에** 수행됩니다.

### 한 행의 여러 수집 방법 <a id="multiple-collection-methods-in-a-row"></a>

`->all()` 또는 `->get()` 모든 결과를 쿼리하면 동일한 결과에 대해 다양한 Collection 작업을 수행 할 수 있으며 매번 데이터베이스를 쿼리하지 않습니다.

```php
$users = User::all();
echo 'Max ID: ' . $users->max('id');
echo 'Average age: ' . $users->avg('age');
echo 'Total budget: ' . $users->sum('budget');
```

### 페이이지네이션 합계 계산 <a id="calculate-sum-with-pagination"></a>

PAGINATED 컬렉션 만있는 경우 모든 레코드의 합계를 계산하는 방법은 무엇입니까? 동일한 쿼리에서 페이지네이션 전에 계산을 실행하십시오.

```php
// How to get sum of post_views with pagination?
$posts = Post::paginate(10);
// This will be only for page 1, not ALL posts
$sum = $posts->sum('post_views');

// Do this with Query Builder
$query = Post::query();
// Calculate sum
$sum = $query->sum('post_views');
// And then do the pagination from the same query
$posts = $query->paginate(10);
```

## 인증 <a id="auth"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (컬렉션)](#collections) ➡️ [다음 (메일)](#mail)

- [한 번에 여러 권한 확인](#check-multiple-permissions-at-once)
- [사용자 등록에 대한 추가 이벤트](#more-events-on-user-registration)
- [Auth::once()에 대해 알고 계셨습니까?](#did-you-know-about-authonce)
- [사용자 비밀번호 업데이트시 API 토큰 변경](#change-api-token-on-users-password-update)
- [최고 관리자에 대한 권한 재정의](#override-permissions-for-super-admin)

### 한 번에 여러 권한 확인 <a id="check-multiple-permissions-at-once"></a>

`@can` Blade 지시문 외에도 `@canany` 지시문으로 한 번에 여러 권한을 확인할 수 있다는 것을 알고 계셨습니까?

```blade
@canany(['update', 'view', 'delete'], $post)
    // The current user can update, view, or delete the post
@elsecanany(['create'], \App\Post::class)
    // The current user can create a post
@endcanany
```

### 사용자 등록에 대한 추가 이벤트 <a id="more-events-on-user-registration"></a>

신규 사용자 등록 후 몇 가지 작업을 수행하고 싶으십니까? `app/Providers/EventServiceProvider.php` 하여 Listeners 클래스를 더 추가 한 다음 해당 클래스에서 `$event->user` 객체로 `handle()` 메서드를 구현합니다.

```php
class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        Registered::class => [
            SendEmailVerificationNotification::class,

            // You can add any Listener class here
            // With handle() method inside of that class
        ],
    ];
```

### Auth::once()에 대해 알고 계셨습니까? <a id="did-you-know-about-authonce"></a>

`Auth::once()` 메소드를 사용하여 ONE REQUEST에 대해서만 사용자로 로그인 할 수 있습니다. 세션이나 쿠키가 사용되지 않으므로 이 방법은 상태를 저장하지 않는 API를 구축 할 때 유용 할 수 있습니다.

```php
if (Auth::once($credentials)) {
    //
}
```

### 사용자 비밀번호 업데이트시 API 토큰 변경 <a id="change-api-token-on-users-password-update"></a>

비밀번호가 변경되면 사용자의 API 토큰을 변경하는 것이 편리합니다.

모델:

```php
public function setPasswordAttribute($value)
{
    $this->attributes['password'] = $value;
    $this->attributes['api_token'] = Str::random(100);
}
```

### 최고 관리자에 대한 권한 재정의 <a id="override-permissions-for-super-admin"></a>

Gates를 정의했지만 SUPER ADMIN 사용자의 모든 권한을 재정의하려는 경우 해당 superadmin ALL 권한을 부여하려면 `AuthServiceProvider.php` 파일에서 `Gate::before()`문을 사용하여 게이트를 가로 챌 수 있습니다.

```php
// Intercept any Gate and check if it's super admin
Gate::before(function($user, $ability) {
    if ($user->is_super_admin == 1) {
        return true;
    }
});

// Or if you use some permissions package...
Gate::before(function($user, $ability) {
    if ($user->hasPermission('root')) {
        return true;
    }
});
```

## 메일 <a id="mail"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (인증)](#auth) ➡️ [다음 (Artisan)](#artisan)

- [laravel.log로 이메일 테스트](#testing-email-into-laravellog)
- [이메일 미리보기](#preview-mailables)
- [Laravel 알림의 기본 이메일 제목](#default-email-subject-in-laravel-notifications)
- [누구에게나 알림 보내기](#send-notifications-to-anyone)

### laravel.log로 이메일 테스트 <a id="testing-email-into-laravellog"></a>

앱에서 이메일 콘텐츠를 테스트하고 싶지만 Mailgun과 같은 설정을 할 수 없거나 설정하지 않으려는 경우 `.env` 매개 변수 `MAIL_DRIVER=log` 를 사용하면 모든 이메일이 실제로 전송되는 대신 `storage/logs/laravel.log` 파일에 저장됩니다. .

### 이메일 미리보기 <a id="preview-mailables"></a>

Mailables를 사용하여 이메일을 보내는 경우 브라우저에서 직접 보내지 않고 결과를 미리 볼 수 있습니다. 경로 결과로 Mailable을 반환하십시오.

```php
Route::get('/mailable', function () {
    $invoice = App\Invoice::find(1);
    return new App\Mail\InvoicePaid($invoice);
});
```

### Laravel 알림의 기본 이메일 제목 <a id="default-email-subject-in-laravel-notifications"></a>

Laravel 알림을 보내고 **toMail()에** subject를 지정하지 않으면 기본 제목은 알림 클래스 이름 인 CamelCased into Spaces입니다.

따라서 다음과 같은 경우 :

```php
class UserRegistrationEmail extends Notification {
    //
}
```

그러면 제목이 **User Registration Email**인 이메일을 받게됩니다.

### 누구에게나 알림 보내기 <a id="send-notifications-to-anyone"></a>

`$user->notify()`를 사용하여 특정 사용자에게뿐만 아니라 `Notification::route()`을 통해 원하는 모든 사람에게 소위 "주문형" Laravel 알림을 보낼 수 있습니다.

```php
Notification::route('mail', 'taylor@example.com')
        ->route('nexmo', '5555555555')
        ->route('slack', 'https://hooks.slack.com/services/...')
        ->notify(new InvoicePaid($invoice));
```

## Artisan <a id="artisan"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (메일)](#mail) ➡️ [다음 (팩토리)](#factories)

- [Artisan 명령 매개 변수](#artisan-command-parameters)
- [유지보수 모드](#maintenance-mode)
- [Artisan 명령 도움말](#artisan-command-help)
- [정확한 Laravel 버전](#exact-laravel-version)
- [어디서나 Artisan 명령 실행](#launch-artisan-command-from-anywhere)

### Artisan 명령 매개 변수 <a id="artisan-command-parameters"></a>

Artisan 명령을 생성 할 때 다양한 방법으로 입력을 요청할 수 있습니다 : `$this->confirm()` , `$this->anticipate()` , `$this->choice()` .

```php
// Yes or no?
if ($this->confirm('Do you wish to continue?')) {
    //
}

// Open question with auto-complete options
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

// One of the listed options with default index
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);
```

### 유지보수 모드 <a id="maintenance-mode"></a>

유지보수 모드를 활성화하려면 artisan down 명령을 실행하십시오.

```bash
php artisan down
```

그러면 사람들은 기본 503 상태 페이지를 보게됩니다.

라라벨 8에서 플래그를 제공 할 수도 있습니다.

- 사용자가 리디렉션되어야하는 경로
- 미리 렌더링되어야하는 뷰
- 유지 관리 모드를 우회하는 비밀 문구
- 유지 관리 모드 중 상태 코드
- X 초마다 페이지 다시로드 재시도

```bash
php artisan down --redirect="/" --render="errors::503" --secret="1630542a-246b-4b66-afa1-dd72a4c43515" --status=200 --retry=60
```

라라벨 8 이전 :

- 표시 될 메시지
- X 초마다 페이지 다시로드 재시도
- 여전히 일부 IP 주소에 대한 액세스 허용

```bash
php artisan down --message="Upgrading Database" --retry=60 --allow=127.0.0.1
```

유지보수 작업을 마쳤으면 다음을 실행하십시오.

```bash
php artisan up
```

### Artisan 명령 도움말 <a id="artisan-command-help"></a>

artisan 명령의 옵션을 확인하려면 `--help` 플래그로 artisan 명령을 실행하십시오. 예를 들어, `php artisan make:model --help` 및 몇 가지 옵션이 있는지 확인하십시오.

```
Options:
  -a, --all             Generate a migration, seeder, factory, and resource controller for the model
  -c, --controller      Create a new controller for the model
  -f, --factory         Create a new factory for the model
      --force           Create the class even if the model already exists
  -m, --migration       Create a new migration file for the model
  -s, --seed            Create a new seeder file for the model
  -p, --pivot           Indicates if the generated model should be a custom intermediate table model
  -r, --resource        Indicates if the generated controller should be a resource controller
      --api             Indicates if the generated controller should be an API controller
  -h, --help            Display this help message
  -q, --quiet           Do not output any message
  -V, --version         Display this application version
      --ansi            Force ANSI output
      --no-ansi         Disable ANSI output
  -n, --no-interaction  Do not ask any interactive question
      --env[=ENV]       The environment the command should run under
  -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
```

### 정확한 Laravel 버전 <a id="exact-laravel-version"></a>

`php artisan --version` 명령을 실행하여 앱에있는 Laravel 버전을 정확히 확인하십시오.

### 어디서나 Artisan 명령 실행 <a id="launch-artisan-command-from-anywhere"></a>

Artisan 명령의 경우 터미널 뿐만 아니라  Artisan::call() 메서드 사용하여 매개 변수와 함께 코드의 어느 곳에서나 실행할 수 있습니다.

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

## 팩토리 <a id="factories"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (Artisan)](#artisan) ➡️ [다음 (로그 및 디버그)](#log-and-debug)

- [팩토리 콜백](#factory-callbacks)
- [Seeder / 팩토리로 이미지 생성](#generate-images-with-seedsfactories)

### 팩토리 콜백 <a id="factory-callbacks"></a>

데이터를 시드하기 위해 팩토리를 사용하는 동안 레코드 삽입 후 일부 작업을 수행하는 팩토리 콜백 기능을 사용 할 수 있습니다.

```php
$factory->afterCreating(App\User::class, function ($user, $faker) {
    $user->accounts()->save(factory(App\Account::class)->make());
});
```

### Seeder / 팩토리로 이미지 생성 <a id="generate-images-with-seedsfactories"></a>

Faker가 텍스트 값뿐만 아니라 이미지도 생성 할 수 있다는 것을 알고 계셨습니까? 아래의 `avatar` 필드를 참고해보세요. 50x50 이미지가 생성됩니다.

```php
$factory->define(User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => bcrypt('password'),
        'remember_token' => Str::random(10),
        'avatar' => $faker->image(storage_path('images'), 50, 50)
    ];
});
```

## 로그 및 디버그 <a id="log-and-debug"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (팩토리)](#factories) ➡️ [다음 (API)](#api)

- [매개 변수로 로깅](#logging-with-parameters)
- [더 편리한 DD](#more-convenient-dd)

### 매개 변수로 로깅 <a id="logging-with-parameters"></a>

`Log::info()` 또는 추가 매개 변수가있는 더 짧은 `info()` 메시지를 작성하여 발생한 상황에 대한 자세한 내용을 볼 수 있습니다.

```php
Log::info('User failed to login.', ['id' => $user->id]);
```

### 더 편리한 DD <a id="more-convenient-dd"></a>

`dd($result)` 를 수행하는 대신 Eloquent 문장이나 컬렉션의 끝에 직접 `->dd()` 를 메소드로 넣을 수 있습니다.

```php
// 이것 대신
$users = User::where('name', 'Taylor')->get();
dd($users);
// 이렇게 할 수 있습니다
$users = User::where('name', 'Taylor')->get()->dd();
```

## API <a id="api"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (로그 및 디버그)](#log-and-debug) ➡️ [다음 (기타)](#other)

- [API 리소스 : "데이터"유무?](#api-resources-with-or-without-data)
- [API에서 "모든 것이 정상입니다"를 반환하기](#api-return-everything-went-ok)

### API 리소스 : "데이터"유무? <a id="api-resources-with-or-without-data"></a>

Eloquent API 리소스를 사용하여 데이터를 반환하면 자동으로 '데이터'에 래핑됩니다. 제거하려면 `app/Providers/AppServiceProvider.php`에 `JsonResource::withoutWrapping();` 를 보면 됩니다

```php
class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        JsonResource::withoutWrapping();
    }
}
```

### API에서 "모든 것이 정상입니다"를 반환하기 <a id="api-return-everything-went-ok"></a>

일부 작업을 수행하지만 응답이 없는 API 엔드포인트라서 "모든 것이 정상입니다"만 반환하려는 경우 204 상태 코드 "No content"를 반환 할 수 있습니다. 라라벨에서는 이것을 쉽게 처리할 수 있습니다. `return response()->noContent();`

```php
public function reorder(Request $request)
{
    foreach ($request->input('rows', []) as $row) {
        Country::find($row['id'])->update(['position' => $row['position']]);
    }

    return response()->noContent();
}
```

## 기타 <a id="other"></a>

⬆️ [맨 위로 이동](#laravel-tips) ⬅️ [이전 (API)](#api)

- [.env의 로컬 호스트](#localhost-in-env)
- ["컴포저 업데이트"를 실행할 때 (또는 실행하지 않을 때)](#when-not-to-run-composer-update)
- [Composer : 최신 버전 확인](#composer-check-for-newer-versions)
- [자동 대문자 번역](#auto-capitalize-translations)
- [시간만 사용하는 카본(Carbon)](#carbon-with-only-hours)
- [단일 액션 컨트롤러](#single-action-controllers)
- [특정 컨트롤러 메서드로 이동](#redirect-to-specific-controller-method)
- [이전 Laravel 버전 사용](#use-older-laravel-version)
- [페이지네이션 링크에 매개 변수 추가](#add-parameters-to-pagination-links)
- [반복 가능한 콜백 함수](#repeatable-callback-functions)
- [리퀘스트 : hasAny](#request-has-any)
- [간단한 페이지네이션](#simple-pagination)

### .env의 로컬 호스트 <a id="localhost-in-env"></a>

`.env` 파일의 `APP_URL` 을 `http://localhost` 에서 실제 URL로 변경하는 것을 잊지 마십시오. 이메일 알림 및 다른 곳에 있는 모든 링크의 기반이됩니다.

```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:9PHz3TL5C4YrdV6Gg/Xkkmx9btaE93j7rQTUZWm2MqU=
APP_DEBUG=true
APP_URL=http://localhost
```

### "컴포저 업데이트"를 실행할 때 (또는 실행하지 않을 때) <a id="when-not-to-run-composer-update"></a>

Laravel에 관해 중요한 내용은 아니지만... 프로덕션 라이브 서버에서 `composer update` 를 실행하지 마십시오. 느리고 저장소가 "깨질"것입니다. 항상 `composer update` 컴퓨터에서 로컬로 실행하고 새 `composer.lock` 을 리포지토리에 커밋하고 라이브 서버에서는 `composer install` 를 실행하십시오.

### Composer : 최신 버전 확인 <a id="composer-check-for-newer-versions"></a>

`composer.json` 패키지 중 어떤 패키지가 최신 버전을 출시했는지 확인하려면 `composer outdated` 실행하십시오. 다음과 같은 모든 정보가 포함 된 전체 목록이 표시됩니다.

```
phpdocumentor/type-resolver 0.4.0 0.7.1
phpunit/php-code-coverage   6.1.4 7.0.3 Library that provides collection, processing, and rende...
phpunit/phpunit             7.5.9 8.1.3 The PHP Unit Testing framework.
ralouphie/getallheaders     2.0.5 3.0.3 A polyfill for getallheaders.
sebastian/global-state      2.0.0 3.0.0 Snapshotting of global state
```

### 자동 대문자 번역 <a id="auto-capitalize-translations"></a>

번역 파일 ( `resources/lang` )에서 변수를 `:variable` 로 지정할 수있을 뿐만 아니라 `:VARIABLE` 또는 `:Variable` 로 대문자로 지정할 수도 있습니다. 그러면 전달하는 값도 자동으로 대문자로 표시됩니다.

```php
// resources/lang/en/messages.php
'welcome' => 'Welcome, :Name'

// Result: "Welcome, Taylor"
echo __('messages.welcome', ['name' => 'taylor']);
```

### 시간만 사용하는 카본(Carbon) <a id="carbon-with-only-hours"></a>

초 또는 분없이 현재 날짜를 보려면 `setSeconds(0)` 또는 `setMinutes(0)`과 같은 Carbon의 메서드를 사용하세요.

```php
// 2020-04-20 08:12:34
echo now();

// 2020-04-20 08:12:00
echo now()->setSeconds(0);

// 2020-04-20 08:00:00
echo now()->setSeconds(0)->setMinutes(0);

// Another way - even shorter
echo now()->startOfHour();
```

### 단일 액션 컨트롤러 <a id="single-action-controllers"></a>

하나의 액션으로 컨트롤러를 만들고 싶다면 `__invoke()` 메서드를 사용하고 "invokable"컨트롤러를 만들 수도 있습니다.

라우트:

```php
Route::get('user/{id}', 'ShowProfile');
```

Artisan:

```bash
php artisan make:controller ShowProfile --invokable
```

컨트롤러:

```php
class ShowProfile extends Controller
{
    public function __invoke($id)
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

### 특정 컨트롤러 메서드로 이동 <a id="redirect-to-specific-controller-method"></a>

URL이나 특정 경로뿐만 아니라 특정 컨트롤러의 특정 메서드로 `redirect()` 하고 매개 변수를 전달할 수도 있습니다. 이것을 사용하십시오 :

```php
return redirect()->action('SomeController@method', ['param' => $value]);
```

### 이전 Laravel 버전 사용 <a id="use-older-laravel-version"></a>

최신 Laravel 대신 OLDER 버전을 사용하려면 다음 명령을 사용하십시오.

```
composer create-project --prefer-dist laravel/laravel project "7.*"
```

*를 원하는 버전으로 변경하십시오.

### 페이지네이션 링크에 매개 변수 추가 <a id="add-parameters-to-pagination-links"></a>

기본 페이지네이션 링크에서 추가 매개 변수를 전달하거나 원래 쿼리 문자열을 보존하거나 특정 `#xxxxx` 앵커를 가리킬 수도 있습니다.

```
{{ $users->appends(['sort' => 'votes'])->links() }}

{{ $users->withQueryString()->links() }}

{{ $users->fragment('foo')->links() }}
```

### 반복 가능한 콜백 함수 <a id="repeatable-callback-functions"></a>

여러 번 재사용해야하는 콜백 함수가 있는 경우 변수에 할당 한 다음 다시 사용할 수 있습니다.

```php
$userCondition = function ($query) {
    $query->where('user_id', auth()->id());
};

// 이 사용자의 댓글이 있는 게시글 가져 오기
// 그리고 이 사용자의 댓글 만 반환
$articles = Article::with(['comments' => $userCondition])
    ->whereHas('comments', $userCondition)
    ->get();
```

### 리퀘스트 : hasAny <a id="request-has-any"></a>

`$request->has()` 메서드로 하나의 매개 변수를 확인할 수있을 뿐만 아니라 `$request->hasAny()`를 사용하여 여러 매개 변수가 있는지 확인할 수도 있습니다.

```php
public function store(Request $request)
{
    if ($request->hasAny(['api_key', 'token'])) {
        echo 'We have API key passed';
    } else {
        echo 'No authorization parameter';
    }
}
```

### 간단한 페이지네이션 <a id="simple-pagination"></a>

페이지네이션에서 모든 페이지 번호 대신 "이전 / 다음"링크 만 갖고 싶다면 (그리고 그로 인해 DB 쿼리 수가 더 적길 바라면) `paginate()`대신 `simplePaginate()` 를 사용하세요.

```php
// Instead of
$users = User::paginate(10);

// You can do this
$users = User::simplePaginate(10);
```
