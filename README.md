# Practical 10: SQLite Database CRUD Operations

[← Back to Index](README.md)

## Database and Networking

### 10. Develop an Android Application Implementing SQLite Database CRUD Operations

**Project Name:** `SQLLiteCRUDDemo`

## Aim

To develop an Android application that performs CRUD (Create, Read, Update, and Delete) operations using an SQLite database.

## Software Requirements

- Android Studio
- Java
- XML
- SQLite Database
- Android SDK

## Project Structure

```text
SQLLiteCRUDDemo/
├── app/
│   └── src/
│       └── main/
│           ├── java/
│           │   └── com/
│           │       └── example/
│           │           └── sqllitecruddemo/
│           │               ├── MainActivity.java
│           │               └── DBHelper.java
│           ├── res/
│           │   └── layout/
│           │       └── activity_main.xml
│           └── AndroidManifest.xml
└── README.md
```

## 1. File Name: `activity_main.xml`

**Location:** `app/src/main/res/layout/activity_main.xml`

This XML file defines the user interface with input fields and buttons for inserting, viewing, updating, and deleting student names.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp">

    <EditText
        android:id="@+id/etName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter Name" />

    <EditText
        android:id="@+id/etNewName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter New Name" />

    <Button
        android:id="@+id/btnInsert"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Insert" />

    <Button
        android:id="@+id/btnView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="View Data" />

    <Button
        android:id="@+id/btnUpdate"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Update" />

    <Button
        android:id="@+id/btnDelete"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Delete" />

    <TextView
        android:id="@+id/txtResult"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:paddingTop="20dp" />

</LinearLayout>
```

## 2. File Name: `DBHelper.java`

**Location:** `app/src/main/java/com/example/sqllitecruddemo/DBHelper.java`

This database helper class creates the SQLite database and provides methods to insert, view, update, and delete student records.

```java
package com.example.sqllitecruddemo;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class DBHelper extends SQLiteOpenHelper {

    public DBHelper(Context context) {
        super(context, "StudentDB", null, 1);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE student(name TEXT)");
    }

    @Override
    public void onUpgrade(
            SQLiteDatabase db,
            int oldVersion,
            int newVersion) {

        db.execSQL("DROP TABLE IF EXISTS student");
        onCreate(db);
    }

    // Insert
    public void insertData(String name) {
        SQLiteDatabase db = this.getWritableDatabase();

        ContentValues values = new ContentValues();
        values.put("name", name);

        db.insert("student", null, values);
    }

    // View
    public String viewData() {
        SQLiteDatabase db = this.getReadableDatabase();

        Cursor cursor = db.rawQuery(
                "SELECT * FROM student",
                null
        );

        StringBuilder data = new StringBuilder();

        try {
            while (cursor.moveToNext()) {
                data.append(cursor.getString(0)).append("\n");
            }
        } finally {
            cursor.close();
        }

        return data.toString();
    }

    // Update
    public void updateData(String oldName, String newName) {
        SQLiteDatabase db = this.getWritableDatabase();

        ContentValues values = new ContentValues();
        values.put("name", newName);

        db.update(
                "student",
                values,
                "name=?",
                new String[]{oldName}
        );
    }

    // Delete
    public void deleteData(String name) {
        SQLiteDatabase db = this.getWritableDatabase();

        db.delete(
                "student",
                "name=?",
                new String[]{name}
        );
    }
}
```

## 3. File Name: `MainActivity.java`

**Location:** `app/src/main/java/com/example/sqllitecruddemo/MainActivity.java`

This Java file connects the user interface to the database helper class and handles button clicks.

```java
package com.example.sqllitecruddemo;

import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    private EditText etName, etNewName;
    private TextView txtResult;
    private DBHelper dbHelper;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        etName = findViewById(R.id.etName);
        etNewName = findViewById(R.id.etNewName);
        txtResult = findViewById(R.id.txtResult);

        Button btnInsert = findViewById(R.id.btnInsert);
        Button btnView = findViewById(R.id.btnView);
        Button btnUpdate = findViewById(R.id.btnUpdate);
        Button btnDelete = findViewById(R.id.btnDelete);

        dbHelper = new DBHelper(this);

        // Insert
        btnInsert.setOnClickListener(v -> {
            String name = etName.getText().toString().trim();

            if (name.isEmpty()) {
                showMessage("Enter a name first");
                return;
            }

            dbHelper.insertData(name);
            showMessage("Data inserted");
            etName.setText("");
        });

        // View
        btnView.setOnClickListener(v -> {
            String data = dbHelper.viewData();

            txtResult.setText(
                    data.isEmpty() ? "No records found" : data
            );
        });

        // Update
        btnUpdate.setOnClickListener(v -> {
            String oldName = etName.getText().toString().trim();
            String newName = etNewName.getText().toString().trim();

            if (oldName.isEmpty() || newName.isEmpty()) {
                showMessage("Enter old and new names");
                return;
            }

            dbHelper.updateData(oldName, newName);
            showMessage("Update completed for matching name(s)");

            etName.setText("");
            etNewName.setText("");
        });

        // Delete
        btnDelete.setOnClickListener(v -> {
            String name = etName.getText().toString().trim();

            if (name.isEmpty()) {
                showMessage("Enter a name to delete");
                return;
            }

            dbHelper.deleteData(name);
            showMessage("Delete completed for matching name(s)");
            etName.setText("");
        });
    }

    private void showMessage(String message) {
        Toast.makeText(
                this,
                message,
                Toast.LENGTH_SHORT
        ).show();
    }

    @Override
    protected void onDestroy() {
        if (dbHelper != null) {
            dbHelper.close();
        }

        super.onDestroy();
    }
}
```

## 4. AndroidManifest.xml

**Location:** `app/src/main/AndroidManifest.xml`

Ensure that the application declares `MainActivity` as its launcher activity. Keep any theme and other settings generated by Android Studio.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:allowBackup="true"
        android:label="SQLite CRUD Demo"
        android:supportsRtl="true"
        android:theme="@style/Theme.SQLLiteCRUDDemo">

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

        </activity>

    </application>

</manifest>
```

**Note:** If your project uses a different theme name, replace `@style/Theme.SQLLiteCRUDDemo` with the existing theme from your project’s `res/values/themes.xml`. Do not replace your existing manifest blindly if it contains other required declarations.

## 5. CRUD Operations Explained

| Operation | Method | Description |
|---|---|---|
| Create | `insertData()` | Inserts a student name into the database. |
| Read | `viewData()` | Retrieves and displays all student names. |
| Update | `updateData()` | Replaces matching old names with a new name. |
| Delete | `deleteData()` | Deletes records matching the entered name. |

## 6. How to Run the Application

1. Open Android Studio.
2. Create a Java project named `SQLLiteCRUDDemo`.
3. Use the package name `com.example.sqllitecruddemo`.
4. Add the XML layout and Java files in their specified locations.
5. Check the Android manifest and theme name.
6. Run the application on an emulator or Android device.

## 7. Sample Output

The application displays two input fields and four buttons:

```text
        SQLite CRUD Demo

    Enter Name
    Enter New Name

        [ Insert ]
        [ View Data ]
        [ Update ]
        [ Delete ]

    Student names appear here
```

### Example Test

1. Enter `Rahul` in the first field and click **Insert**.
2. Click **View Data** to display `Rahul`.
3. Enter `Rahul` in the first field and `Arun` in the second field. Click **Update**.
4. Click **View Data** to confirm that `Arun` is displayed instead of `Rahul`.
5. Enter `Arun` in the first field and click **Delete**.
6. Click **View Data** again. The application displays `No records found` if the database is empty.

## 8. Important Notes

- SQLite stores the data locally on the device.
- The database is named `StudentDB`, and its table is named `student`.
- The current table stores only student names, not unique student IDs.
- If duplicate names exist, the update and delete operations affect **all records matching that name**.
- The `onUpgrade()` method drops the existing table. If the database version is increased, existing records will be lost. A production application should use a proper database migration instead.
- No internet permission is required for these local database operations.

## Result

Thus, an Android application implementing SQLite database CRUD operations was developed successfully. The application can insert, retrieve, update, and delete student names using a local SQLite database.

## 9. Upload to GitHub

Run the following commands in the project directory if Git is already initialized and the remote repository is configured:

```bash
git add .
git commit -m "Add SQLite CRUD operations practical"
git push
```

If this is a new repository, create the GitHub repository and configure its remote URL before pushing.
