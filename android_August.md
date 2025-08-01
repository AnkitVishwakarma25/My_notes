## ✅ Part 1: ViewModel with Navigation in Jetpack Compose

Let’s build a simple example where:

- You navigate from `ScreenA` to `ScreenB`

- Data typed in `ScreenA` is **retained and shown** in `ScreenB` using the **same ViewModel**

---

### 🔧 Step-by-Step (Jetpack Compose)

#### 1. Dependencies in `build.gradle` (Compose + Navigation)

gradle

CopyEdit

`implementation "androidx.navigation:navigation-compose:2.7.5" implementation "androidx.lifecycle:lifecycle-viewmodel-compose:2.7.0"`

---

#### 2. ViewModel

kotlin

CopyEdit

`class SharedViewModel : ViewModel() {    private val _name = MutableStateFlow("")    val name: StateFlow<String> = _name    fun updateName(newName: String) {         _name.value = newName     } }`

---

#### 3. Navigation Setup

kotlin

CopyEdit

`@Composable fun NavGraph(navController: NavHostController, sharedViewModel: SharedViewModel) {     NavHost(navController, startDestination = "screenA") {         composable("screenA") {             ScreenA(navController, sharedViewModel)         }         composable("screenB") {             ScreenB(sharedViewModel)         }     } }`

---

#### 4. ScreenA

kotlin

CopyEdit

`@Composable fun ScreenA(navController: NavController, sharedViewModel: SharedViewModel) {    val name by sharedViewModel.name.collectAsState()     Column {         TextField(             value = name,             onValueChange = { sharedViewModel.updateName(it) },             label = { Text("Enter name") }         )         Button(onClick = { navController.navigate("screenB") }) {             Text("Go to Screen B")         }     } }`

---

#### 5. ScreenB

kotlin

CopyEdit

`@Composable fun ScreenB(sharedViewModel: SharedViewModel) {    val name by sharedViewModel.name.collectAsState()     Column {         Text(text = "You typed: $name")     } }`

---

#### 6. MainActivity

kotlin

CopyEdit

`class MainActivity : ComponentActivity() {    override fun onCreate(savedInstanceState: Bundle?) {        super.onCreate(savedInstanceState)         setContent {            val navController = rememberNavController()            val sharedViewModel: SharedViewModel = viewModel()             NavGraph(navController, sharedViewModel)         }     } }`

---

## ✅ Part 2: ViewModel with Navigation in XML-based Apps

In **XML + Fragment Navigation**, the same concept applies:

### 📌 Use `ViewModelProvider(requireActivity())` in fragments to share a ViewModel.

---

#### 1. ViewModel

kotlin

CopyEdit

`class SharedViewModel : ViewModel() {    val name = MutableLiveData<String>() }`

---

#### 2. Fragment A (input)

kotlin

CopyEdit

`class FragmentA : Fragment() {    private lateinit var viewModel: SharedViewModel    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {         viewModel = ViewModelProvider(requireActivity()).get(SharedViewModel::class.java)        val editText = view.findViewById<EditText>(R.id.editText)        val button = view.findViewById<Button>(R.id.button)         button.setOnClickListener {             viewModel.name.value = editText.text.toString()             findNavController().navigate(R.id.action_fragmentA_to_fragmentB)         }     } }`

---

#### 3. Fragment B (output)

kotlin

CopyEdit

`class FragmentB : Fragment() {    private lateinit var viewModel: SharedViewModel    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {         viewModel = ViewModelProvider(requireActivity()).get(SharedViewModel::class.java)        val textView = view.findViewById<TextView>(R.id.textView)         viewModel.name.observe(viewLifecycleOwner) {             textView.text = it         }     } }`

---

## 📌 Summary

| Platform        | How to Share ViewModel                                        |
| --------------- | ------------------------------------------------------------- |
| Jetpack Compose | Create ViewModel in `Activity` or `NavGraph`, pass to screens |
| XML + Fragments | Use `ViewModelProvider(requireActivity())`                    |

## 🔐 What is `SavedStateHandle`?

`SavedStateHandle` is a lifecycle-aware key-value map used in `ViewModel` to:

- Automatically **save and restore state** (like a Bundle).

- Restore data after process death or screen rotation.

- Works well with both **Jetpack Compose** and **XML**.

---

## ✅ When to Use It?

- You want to **restore values** (e.g., input fields, screen state) **after the app is killed in background and relaunched**.

- Example: user types their name, system kills app, user comes back and sees the name is still there.

---

## ✅ Add Dependency

Add this if not already in `build.gradle`:

gradle

CopyEdit

`implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:2.7.0"`

---

## 🔧 Step-by-Step: Jetpack Compose Example

### 🧠 1. ViewModel with `SavedStateHandle`

kotlin

CopyEdit

`class MyViewModel(    private val savedStateHandle: SavedStateHandle ) : ViewModel() {    // Key for storage     companion object {        const val KEY_NAME = "user_name"     }    // Backed by savedStateHandle     private val _name = MutableStateFlow(savedStateHandle.get<String>(KEY_NAME) ?: "")    val name: StateFlow<String> = _name    fun updateName(newName: String) {         _name.value = newName         savedStateHandle[KEY_NAME] = newName // Save to survive process death     } }`

---

### 🧩 2. Provide ViewModel with `SavedStateHandle`

kotlin

CopyEdit

`@Composable fun MyScreen() {    val viewModel: MyViewModel = viewModel() // Automatically injects SavedStateHandle     val name by viewModel.name.collectAsState()     Column {         TextField(             value = name,             onValueChange = viewModel::updateName,             label = { Text("Enter name") }         )         Text("You typed: $name")     } }`

---

✅ Now even if the app is **killed in background** and relaunched, the value will be restored!

---

## 🔧 XML-Based App Example

### 1. ViewModel with SavedStateHandle

kotlin

CopyEdit

`class MyViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {    companion object {        const val KEY_NAME = "user_name"     }    val name: MutableLiveData<String> =         savedStateHandle.getLiveData(KEY_NAME)    fun updateName(newName: String) {         name.value = newName         savedStateHandle[KEY_NAME] = newName     } }`

---

### 2. Provide ViewModel in Activity or Fragment

kotlin

CopyEdit

`class MainActivity : AppCompatActivity() {    private val viewModel: MyViewModel by viewModels() // SavedStateHandle automatically provided      override fun onCreate(savedInstanceState: Bundle?) {        super.onCreate(savedInstanceState)         setContentView(R.layout.activity_main)        val editText = findViewById<EditText>(R.id.editText)        val textView = findViewById<TextView>(R.id.textView)         viewModel.name.observe(this) {             editText.setText(it)             textView.text = "Hello $it"         }         editText.addTextChangedListener {             viewModel.updateName(it.toString())         }     } }`

---

## 🧪 Test It:

1. Type something in the field.

2. Press Home → Remove app from recent → Open app again.

3. The input value will still be there!

---

## 📝 Summary

| Term                            | Purpose                                               |
| ------------------------------- | ----------------------------------------------------- |
| `SavedStateHandle`              | Save/restore values in ViewModel                      |
| `savedStateHandle[key] = value` | Save a value                                          |
| `savedStateHandle.get<T>(key)`  | Get a saved value                                     |
| Jetpack Compose                 | Use `viewModel()` — SavedStateHandle is auto-injected |
| XML                             | Use `by viewModels()` — auto-provided by Jetpack      |
