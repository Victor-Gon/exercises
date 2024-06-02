# exercises

## NodeJS

### Exercice: Is there a problem? (1 points)

```	
// Call web service and return count user, (got is library to call url)
async function getCountUsers() {
  return { total: await got.get('https://my-webservice.moveecar.com/users/count') };
}

// Add total from service with 20
async function computeResult() {
  const result = getCountUsers();
  return result.total + 20;
}
```

First of all, the function `getCountUsers` is an async function, so it should be called with `await` keyword.
Otherwise, we would be trying to sum the promise's attribute `total` (undefined) with 20, which would result in `NaN`.
It is also necessary to consider what is returned by `got.get` function, which is probably not a plain number, but a response object with a `data` or `body` attribute that contains the actual number of users.
It will also be a good practice to handle possible errors that may occur as a result of the request to the web service.
```
// Call web service and return count user, (got is library to call url)
async function getCountUsers() {
  try {
    const response = await got.get('https://my-webservice.moveecar.com/users/count');
    return { total: response.data.total };
  } catch (error) {
    console.error('Error fetching user count:', error);
    // Handle error as needed
    return { total: 0 };
  }
}

// Add total from service with 20
async function computeResult() {
  const result = await getCountUsers();
  return result.total + 20;
}

```


### Exercice: Is there a problem? (2 points)
    
```
// Call web service and return total vehicles, (got is library to call url)
async function getTotalVehicles() {
    return await got.get('https://my-webservice.moveecar.com/vehicles/total');
}

function getPlurial() {
    let total;
    getTotalVehicles().then(r => total = r);
    if (total <= 0) {
        return 'none';
    }
    if (total <= 10) {
        return 'few';
    }
    return 'many';
}
```

Similar to our previous exercise, it would be great to add error handling to the `getTotalVehicles` function.
We must again also consider what is returned by `got.get` function, which is probably not a plain number.
The `getPlurial` function is also problematic because it will try to access `total` before it is assigned as
it is not waiting for the promise to resolve. To fix this, we can use `await` keyword to wait for the promise to resolve or use `then` to handle the promise resolution and do the logic and return inside the callback. In both cases, we need to declare `getPlurial` as an async function.
```
// Call web service and return total vehicles, (got is library to call url)
async function getTotalVehicles() {
    try {
        const response = await got.get('https://my-webservice.moveecar.com/vehicles/total');
        return response.data.total;
    } catch (error) {
        console.error('Error fetching total vehicles:', error);
        // Handle error as needed
        return 0;
    }
}

async function getPlurial() {
    return new Promise((resolve, reject) => {
        getTotalVehicles().then(total => {
            if (total <= 0) {
                resolve('none');
            }
            if (total <= 10) {
                resolve('few');
            }
            resolve('many');
        }).catch(error => {
            reject(error);
        });
    });
}
```


### Exercice: Unit test (2 points)

```
import { expect, test } from '@jest/globals';

function getCapitalizeFirstWord(name: string): string {
  if (name == null) {
    throw new Error('Failed to capitalize first word with null');
  }
  if (!name) {
    return name;
  }
  return name.split(' ').map(
    n => n.length > 1 ? (n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()) : n
  ).join(' ');
}

test('capitalize the first letter of each word', () => {
  const result = getCapitalizeFirstWord('hello world');
  expect(result).toBe('Hello World');
});

test('empty string', () => {
  const result = getCapitalizeFirstWord('');
  expect(result).toBe('');
});

test('single character words', () => {
  const result = getCapitalizeFirstWord('a b c');
  expect(result).toBe('A B C');
});

test('mixed case', () => {
  const result = getCapitalizeFirstWord('hElLo WoRlD');
  expect(result).toBe('Hello World');
});

test('same string if already capitalized', () => {
  const result = getCapitalizeFirstWord('Hello World');
  expect(result).toBe('Hello World');
});

test('multiple spaces', () => {
  const result = getCapitalizeFirstWord('hello   world');
  expect(result).toBe('Hello   World');
});

test('error if input is null', () => {
  expect(() => getCapitalizeFirstWord(null as any as string)).toThrow('Failed to capitalize first word with null');
});

test('error if input is undefined', () => {
  expect(() => getCapitalizeFirstWord(undefined as any as string)).toThrow('Failed to capitalize first word with null');
});

test('null string', () => {
  const result = getCapitalizeFirstWord('null');
  expect(result).toBe('Null');
});

test('undefined string', () => {
  const result = getCapitalizeFirstWord('undefined');
  expect(result).toBe('Undefined');
});


// In this case, the function does not throw a custom error but breaks with a TypeError.
// Maybe we should also cover this
test('error if input is not a string', () => {
  expect(() => getCapitalizeFirstWord(123 as any as string)).toThrow(TypeError);
});
```
Notice that the function is called `getCapitalizeFirstWord` but it is not returning the first word capitalized, but all words in the string. The function should probably be renamed or change its implementation.



## Angular

### Exercice: Is there a problem and improve the code (5 points)
```
@Component({
  selector: 'app-users',
  template: `
    <input type="text" [(ngModel)]="query" (ngModelChange)="querySubject.next($event)">
    <div *ngFor="let user of users">
        {{ user.email }}
    </div>
  `
})
export class AppUsers implements OnInit {

  query = '';
  querySubject = new Subject<string>();

  users: { email: string; }[] = [];

  constructor(
    private userService: UserService
  ) {
  }

  ngOnInit(): void {
    concat(
      of(this.query),
      this.querySubject.asObservable()
    ).pipe(
      concatMap(q =>
        timer(0, 60000).pipe(
          this.userService.findUsers(q)
        )
      )
    ).subscribe({
      next: (res) => this.users = res
    });
  }
}
```

First af all, it is a good practice to unsubscribe from observables when the component is destroyed to avoid memory leaks. Same with error handling as we are making a request to a service.
We can use 'distinctUntilChanged' and 'debounceTime', which are RxJS recommended operators to avoid unnecessary requests (avoid fetching with the same query and avoid fetching too often) to our service.
We do not need to use 'concat' and 'concatMap' operators in this case, since we are only interested in the latest value of the query which is emitted by 'querySubject'. Thus, we can use 'switchMap' operator instead as, if
subsequent values are emitted before the previous request is completed, it will cancel the previous request and only return the latest one.
```
@Component({
  selector: 'app-users',
  template: `
    <input type="text" [(ngModel)]="query" (ngModelChange)="querySubject.next($event)">
    <div *ngFor="let user of users">
        {{ user.email }}
    </div>
  `
})
export class AppUsers implements OnInit, OnDestroy {

  query = '';
  querySubject = new Subject<string>();
  private subscription = new Subscription();

  users: { email: string; }[] = [];

  constructor(
    private userService: UserService
  ) {
  }

  ngOnInit(): void {
    this.subscription.add(
      this.querySubject.pipe(
        debounceTime(300),
        distinctUntilChanged(),
        switchMap(q =>
            timer(0, 60000).pipe(
              switchMap(() => this.userService.findUsers(q))
            )
        )
      ).subscribe({
        next: (res) => this.users = res,
        error: (error) => console.error('Error fetching users:', error)
      })
    );
  }

  ngOnDestroy(): void {
    this.subscription.unsubscribe();
  }
}
```


### Exercice: Improve performance (5 points)
```
@Component({
  selector: 'app-users',
  template: `
    <div *ngFor="let user of users">
        {{ getCapitalizeFirstWord(user.name) }}
    </div>
  `
})
export class AppUsers {

  @Input()
  users: { name: string; }[];

  constructor() {}
  
  getCapitalizeFirstWord(name: string): string {
    return name.split(' ').map(n => n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()).join(' ');
  }
}
```

The function `getCapitalizeFirstWord` is called for each user from the template. So it could be called multiple times for the same user on each change detection cycle.
We can improve this by transforming the list of users only when it changes.
```
@Component({
  selector: 'app-users',
  template: `
    <div *ngFor="let user of capitalizedUsers">
        {{ user.name }}
    </div>
  `
})
export class AppUsers {

  @Input()
  users: { name: string; }[];

  capitalizedUsers: { name: string; }[];

  constructor() {}
  
  ngOnChanges(changes: SimpleChanges): void {
    if (changes.users) {
      this.capitalizedUsers = this.users.map(u => ({ name: this.getCapitalizeFirstWord(u.name) }));
    }
  }

  getCapitalizeFirstWord(name: string): string {
    return name.split(' ').map(n => n.substring(0, 1).toUpperCase() + n.substring(1).toLowerCase()).join(' ');
  }
}
```


### Exercice: Forms (8 points)
Complete and modify AppUserForm class to use Angular Reactive Forms. Add a button to submit.
The form should return data in this format
```
{
  email: string; // mandatory, must be a email
  name: string; // mandatory, max 128 characters
  birthday?: Date; // Not mandatory, must be less than today
  address: { // mandatory
    zip: number; // mandatory
    city: string; // mandatory, must contains only alpha uppercase and lower and space
  };
}
```
```
@Component({
  selector: 'app-user-form',
  template: `
    <form>
        <input type="text" placeholder="email">
        <input type="text" placeholder="name">
        <input type="date" placeholder="birthday">
        <input type="number" placeholder="zip">
        <input type="text" placeholder="city">
    </form>
  `
})
export class AppUserForm {

  @Output()
  event = new EventEmitter<{ email: string; name: string; birthday: Date; address: { zip: number; city: string; };}>;
  
  constructor(
    private formBuilder: FormBuilder
  ) {
  }

  doSubmit(): void {
    this.event.emit(...);
  }
}
```

```
@Component({
  selector: 'app-user-form',
  template: `
    <form [formGroup]="userForm" (ngSubmit)="doSubmit()">
        <input type="email" formControlName="email" placeholder="email">
        <input type="text" formControlName="name" placeholder="name">
        <input type="date" formControlName="birthday" placeholder="birthday">
        <input type="number" formControlName="zip" placeholder="zip">
        <input type="text" formControlName="city" placeholder="city">
        <button type="submit" [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class AppUserForm {

  @Output()
  event = new EventEmitter<{
    email: string;
    name: string;
    birthday: Date;
    address: { zip: number; city: string; };
  }>();
  userForm: FormGroup;

  constructor(
    private formBuilder: FormBuilder
  ) {
    this.userForm = this.formBuilder.group({
      email: ['', [Validators.required, Validators.email]],
      name: ['', [Validators.required, Validators.maxLength(128)]],
      birthday: ['', [this.birthdayValidator]],
      address: this.formBuilder.group({
        zip: ['', [Validators.required]],
        city: ['', [Validators.required, Validators.pattern(/^[a-zA-Z ]+$/)]]
      })
    });
  }

  doSubmit(): void {
    if (this.userForm.valid) {
      this.event.emit(this.userForm.value);
    }
  }

  private birthdayValidator(control: AbstractControl): ValidationErrors | null {
    const birthday = control.value;
    if (birthday && new Date(birthday) >= new Date()) {
      return { futureDate: true };
    }
    return null;
  }
}
```


## CSS & Bootstrap

### Exercice: Card (5 points)
```
<!doctype html>
<html lang="en">
  <head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">

    <style>
    .modal-header {
        border-bottom: none;
    }

    .modal-footer {
        border-top: none;
    }

    .notification-icon {
        position: absolute;
        top: -10px;
        right: -10px;
        background-color: red;
        font-weight: bold;
        color: white;
        height: 18px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
    }

    .notification-icon::before {
        content: '';
        position: absolute;
        left: -10px;
        width: 10px;
        height: 18px;
        background-color: red;
        border-top-left-radius: 12.5px;
        border-bottom-left-radius: 12.5px;
    }

    /* Right semicircle */
    .notification-icon::after {
        content: '';
        position: absolute;
        right: -10px;
        width: 10px;
        height: 18px;
        background-color: red;
        border-top-right-radius: 12.5px;
        border-bottom-right-radius: 12.5px;
    }

    .rounded-r-0 {
        border-top-right-radius: 0;
        border-bottom-right-radius: 0;
    }

    .rounded-l-0 {
        border-top-left-radius: 0;
        border-bottom-left-radius: 0;
    }

    .fw-600 {
        font-weight: 600;
    }

    .fw-500 {
        font-weight: 500;
    }

    @media (min-width: 576px) {
        .modal-dialog.maxw-450px {
            max-width: 450px;
            margin: 1.75rem auto;
        }
    }
    </style>

    <title>Hello, world!</title>
  </head>
  <body>
    <h1>Hello, world!</h1>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>

    <!-- Button trigger modal -->
    <button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#exampleModal">
        Open Popup
    </button>

    <!-- Modal -->
    <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered maxw-450px">
        <div class="modal-content">
            <div class="modal-header pb-1">
            <h5 class="modal-title fw-600" id="exampleModalLabel">Exercice</h5>
            <!-- Notification icon -->
            <div class="notification-icon">
                <span class="font-weight-bold text-wihite">+123</span>
            </div>
            </div>
            <div class="modal-body py-0">
            <p class="mb-0 fw-500">Redo this card in css and if possible ussing bootstrap 4 or 5</p>
            </div>
            <div class="modal-footer justify-content-end">
            <div class="d-flex">
                <button type="button" class="btn btn-dark rounded-r-0">Got it</button>
                <button type="button" class="btn btn-outline-dark rounded-l-0">I don't know</button>
            </div>
            <div class="dropdown">
                <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton" data-bs-toggle="dropdown" aria-expanded="false">
                Options
                </button>
                <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton">
                    <li><a class="dropdown-item" href="#">Action</a></li>
                    <li><a class="dropdown-item" href="#">Another action</a></li>
                    <li><a class="dropdown-item" href="#">Something else here</a></li>
                </ul>
            </div>
            </div>
        </div>
    </div>
  </body>
</html>
```


## MongoDB

### Exercice: MongoDb request (3 points)
MongoDb collection users with schema
```
{
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
}
```
Complete the query, you have a variable that contains a piece of text to search for. Search by exact email, starts with first or last name and only users logged in for 6 months
```
db.collections('users').find({
    $or: [
        { email: searchText },
        { first_name: { $regex: `^${searchText}`, $options: 'i' } },
        { last_name: { $regex: `^${searchText}`, $options: 'i' } }
    ],
    last_connection_date: { $gte: new Date(new Date().setMonth(new Date().getMonth() - 6)) }
});
```
What should be added to the collection so that the query is not slow?

We can add several indexes to the collection for the fields used in the query.
```
db.collections('users').createIndex({ email: 1 });
db.collections('users').createIndex({ first_name: 1 });
db.collections('users').createIndex({ last_name: 1 });
db.collections('users').createIndex({ last_connection_date: 1 });
```


### Exercice: MongoDb aggregate (5 points)
MongoDb collection users with schema
```
{
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
}
```
Complete the aggregation so that it sends user emails by role ({_id: 'role', users: [email,...]})

First of all, we need to use `$unwind` operator to deconstruct the `roles` array field so that we can group users
by role instead of exact combination of roles. Then we group documents by role and register their email using
`$addToSet` operator to avoid duplicates.
```
db.collections('users').aggregate([
    { $unwind: '$roles' },
    { $group: { _id: '$roles', emails: { $addToSet : '$email' } } }
]);
```


### Exercice: MongoDb update (5 points)
MongoDb collection users with schema
```
 {
    email: string;
    first_name: string;
    last_name: string;
    roles: string[];
    last_connection_date: Date;
    addresses: {
        zip: number;
        city: string;
    }[]:
}
```
Update document ObjectId("5cd96d3ed5d3e20029627d4a"), modify only last_connection_date with current date
```
db.collections('users').updateOne(
    { _id: ObjectId("5cd96d3ed5d3e20029627d4a") },
    { $set: { last_connection_date: new Date() } }
);
```
Update document ObjectId("5cd96d3ed5d3e20029627d4a"), add a role admin
```
db.collections('users').updateOne(
    { _id: ObjectId("5cd96d3ed5d3e20029627d4a") },
    { $addToSet: { roles: 'admin' } }
);
```
Update document ObjectId("5cd96d3ed5d3e20029627d4a"), modify addresses with zip 75001 and replace city with Paris 1
```
db.collections('users').updateOne(
    { _id: ObjectId("5cd96d3ed5d3e20029627d4a"), 'addresses.zip': 75001 },
    { $set: { 'addresses.$.city': 'Paris 1' } }
);
```