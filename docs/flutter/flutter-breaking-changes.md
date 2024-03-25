---
comments: true
---


## Flatbutton 대신 TextButton 으로 마이그레이션

```dart
new Container(
    child: Flatbutton(    
     color: Colors.blue,
     textColor: Colors.white,
     disabledColor: Colors.grey,
     disabledTextColor: Colors.black,
     padding: EdgeInsets.all(8.0),
     splashColor: Colors.blueAccent,
    onPressed: () {
        /*...*/
    },
    child: Text(
        "Edit Profiles",
        style: TextStyle(fontSize: 18.0),
    ),
  ),
)

new Container(
    child: TextButton(
    style: ButtonStyle(
        foregroundColor: MaterialStateProperty.all<Color>(Colors.red),
    ),
    onPressed: () {
        /*...*/
    },
    child: Text(
        "Edit Profiles",
        style: TextStyle(fontSize: 18.0),
    ),
  ),
)
```

!!! 출처
    * [https://stackoverflow.com/questions/73737240/error-the-method-flatbutton-isnt-defined-for-the-class-platformbutton](https://stackoverflow.com/questions/73737240/error-the-method-flatbutton-isnt-defined-for-the-class-platformbutton)
    * [https://docs.flutter.dev/release/breaking-changes/buttons#migration-guide](https://docs.flutter.dev/release/breaking-changes/buttons#migration-guide)