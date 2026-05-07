# 📱 Lab 19 — ViewModel & LiveData (Android Java)

---

## 🎯 Objectifs

* Comprendre pourquoi une variable classique est perdue lors d’une rotation d’écran.
* Découvrir les limites de `onSaveInstanceState()`.
* Maîtriser **ViewModel** (persistance des données).
* Utiliser **LiveData** pour mettre à jour l’UI automatiquement.
* Comprendre les concepts : LifecycleOwner, Observer, ViewModelStore.
* Appliquer les bonnes pratiques Android modernes (Jetpack 2026).

---

## 📚 Théorie (simple)

Quand on tourne l’écran :

* Android détruit l’Activity (`onDestroy`).
* Il recrée une nouvelle (`onCreate`).
* Toutes les variables sont perdues ❌

### 🔴 Ancienne solution

```java
onSaveInstanceState(Bundle outState)
```

✔ Sauvegarde seulement des types simples
❌ Limité (pas d’objets complexes, pas de logique)

### 🟢 Solution moderne

* **ViewModel** → garde les données
* **LiveData** → met à jour l’UI automatiquement

---

## ⚙️ Création du projet

* Nom : `ViewModelLiveDataDemoEnrichi`
* Language : Java
* Min SDK : API 24

### 📦 Dépendances

```gradle
def lifecycle_version = "2.10.0"

implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
implementation "androidx.lifecycle:lifecycle-livedata:$lifecycle_version"
```

---

# 🧪 Partie 1 — SANS ViewModel (Problème)

## 📄 Layout

```xml
<LinearLayout android:orientation="vertical" android:gravity="center">

    <TextView
        android:id="@+id/tvCount"
        android:text="0"
        android:textSize="80sp" />

    <Button
        android:id="@+id/btnIncrement"
        android:text="INCRÉMENTER" />

    <Button
        android:id="@+id/btnDecrement"
        android:text="DÉCRÉMENTER" />

    <Button
        android:id="@+id/btnReset"
        android:text="RÉINITIALISER" />

</LinearLayout>
```

## 📄 MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private int count = 0; // ❌ perdue à la rotation

    private TextView tvCount;
    private Button btnIncrement, btnDecrement, btnReset;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvCount = findViewById(R.id.tvCount);
        btnIncrement = findViewById(R.id.btnIncrement);
        btnDecrement = findViewById(R.id.btnDecrement);
        btnReset = findViewById(R.id.btnReset);

        if (savedInstanceState != null) {
            count = savedInstanceState.getInt("count_key", 0);
        }

        updateUI();

        btnIncrement.setOnClickListener(v -> { count++; updateUI(); });
        btnDecrement.setOnClickListener(v -> { count--; updateUI(); });
        btnReset.setOnClickListener(v -> { count = 0; updateUI(); });
    }

    private void updateUI() {
        tvCount.setText(String.valueOf(count));
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putInt("count_key", count);
    }
}
```

### ❌ Résultat

* Rotation → compteur revient à 0

---

# ✅ Partie 2 — AVEC ViewModel + LiveData

## 📄 CounterViewModel.java

```java
public class CounterViewModel extends ViewModel {

    private final MutableLiveData<Integer> countLiveData = new MutableLiveData<>();

    public CounterViewModel() {
        countLiveData.setValue(0);
    }

    public void increment() {
        Integer current = countLiveData.getValue();
        if (current != null) {
            countLiveData.setValue(current + 1);
        }
    }

    public void decrement() {
        Integer current = countLiveData.getValue();
        if (current != null) {
            countLiveData.setValue(current - 1);
        }
    }

    public void reset() {
        countLiveData.setValue(0);
    }

    public LiveData<Integer> getCount() {
        return countLiveData;
    }
}
```

---

## 📄 MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private CounterViewModel viewModel;

    private TextView tvCount;
    private Button btnIncrement, btnDecrement, btnReset;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvCount = findViewById(R.id.tvCount);
        btnIncrement = findViewById(R.id.btnIncrement);
        btnDecrement = findViewById(R.id.btnDecrement);
        btnReset = findViewById(R.id.btnReset);

        viewModel = new ViewModelProvider(this).get(CounterViewModel.class);

        viewModel.getCount().observe(this, new Observer<Integer>() {
            @Override
            public void onChanged(Integer newCount) {
                tvCount.setText(String.valueOf(newCount));
            }
        });

        btnIncrement.setOnClickListener(v -> viewModel.increment());
        btnDecrement.setOnClickListener(v -> viewModel.decrement());
        btnReset.setOnClickListener(v -> viewModel.reset());
    }
}
```

---

## 🧪 Tests

* Rotation → compteur conservé ✅
* Mode sombre → OK ✅
* Background thread → OK ✅

---

## 📊 Comparaison

| Critère          | Sans ViewModel | Avec ViewModel |
| ---------------- | -------------- | -------------- |
| Rotation         | ❌              | ✅              |
| UI auto          | ❌              | ✅              |
| Code propre      | ❌              | ✅              |
| Objets complexes | ❌              | ✅              |

---

## 🚀 Bonus

### Thread background

```java
public void incrementFromBackground() {
    new Thread(() -> {
        Integer current = countLiveData.getValue();
        if (current != null) {
            countLiveData.postValue(current + 1);
        }
    }).start();
}
```

---

## 🏁 Conclusion

* ❌ Avant : perte de données + code compliqué
* ✅ Après : données persistantes + UI automatique

👉 ViewModel + LiveData = base de toutes les apps modernes Android

---

💡 Lab réussi si :

* Rotation ne casse plus l’app
* UI se met à jour automatiquement
* Code est séparé (MVVM)
