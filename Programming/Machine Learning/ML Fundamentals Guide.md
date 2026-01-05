# Machine Learning - Fundamentos Completos

> **Guía profesional de Machine Learning con explicaciones conceptuales profundas**
> Última actualización: Enero 2026

---

## Tabla de Contenidos

1. [[#¿Qué es Machine Learning?|Fundamentos de ML]]
2. [[#Tipos de Aprendizaje|Tipos de Aprendizaje]]
3. [[#El Pipeline de Machine Learning|Pipeline de ML]]
4. [[#Preparación de Datos|Preparación de Datos]]
5. [[#Métricas y Evaluación|Métricas y Evaluación]]
6. [[#Overfitting y Underfitting|Overfitting y Underfitting]]

---

## ¿Qué es Machine Learning?

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: MACHINE LEARNING                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Machine Learning es un subcampo de la Inteligencia Artificial  │
│  donde los sistemas APRENDEN de los datos en lugar de ser       │
│  programados explícitamente con reglas.                         │
│                                                                 │
│  PROGRAMACIÓN TRADICIONAL:                                      │
│  ┌─────────┐                                                    │
│  │ Datos   │────┐                                               │
│  └─────────┘    │    ┌──────────────┐    ┌─────────────┐        │
│                 ├───▶│   PROGRAMA   │───▶│   Salida    │        │
│  ┌─────────┐    │    │  (Reglas)    │    └─────────────┘        │
│  │ Reglas  │────┘    └──────────────┘                           │
│  └─────────┘                                                    │
│  Tú escribes las reglas: "Si email contiene 'gratis' → spam"    │
│                                                                 │
│  MACHINE LEARNING:                                              │
│  ┌─────────┐                                                    │
│  │ Datos   │────┐                                               │
│  └─────────┘    │    ┌──────────────┐    ┌─────────────┐        │
│                 ├───▶│  ALGORITMO   │───▶│   MODELO    │        │
│  ┌─────────┐    │    │  (Aprende)   │    │  (Reglas)   │        │
│  │Etiquetas│────┘    └──────────────┘    └─────────────┘        │
│  └─────────┘                                                    │
│  El algoritmo DESCUBRE las reglas a partir de ejemplos          │
│                                                                 │
│  ANALOGÍA:                                                      │
│  • Tradicional: Te dan un libro de cocina con recetas           │
│  • ML: Te dan 1000 platos y debes descubrir las recetas         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ¿Por Qué Machine Learning?

```
┌─────────────────────────────────────────────────────────────────┐
│         CUÁNDO USAR MACHINE LEARNING (Y CUÁNDO NO)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ USAR ML CUANDO:                                             │
│                                                                 │
│  1. Las reglas son demasiado complejas para escribir            │
│     "¿Cómo describes las reglas para reconocer un gato?"        │
│                                                                 │
│  2. Las reglas cambian constantemente                           │
│     "El spam evoluciona, nuevas palabras, nuevas técnicas"      │
│                                                                 │
│  3. Hay patrones ocultos en grandes volúmenes de datos          │
│     "¿Qué clientes van a cancelar su suscripción?"              │
│                                                                 │
│  4. Necesitas personalización a escala                          │
│     "Recomendaciones diferentes para millones de usuarios"      │
│                                                                 │
│  ❌ NO USAR ML CUANDO:                                          │
│                                                                 │
│  1. Las reglas son simples y conocidas                          │
│     "Calcular impuestos" → Usa fórmulas                         │
│                                                                 │
│  2. No tienes suficientes datos                                 │
│     ML necesita ejemplos para aprender                          │
│                                                                 │
│  3. Necesitas 100% de explicabilidad legal                      │
│     "¿Por qué rechazaste mi préstamo?" → Difícil con ML        │
│                                                                 │
│  4. El costo de error es inaceptable                            │
│     Sistemas críticos de vida sin validación humana             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tipos de Aprendizaje

### Vista General

```
┌─────────────────────────────────────────────────────────────────┐
│                 TIPOS DE MACHINE LEARNING                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   MACHINE LEARNING                      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                            │                                    │
│         ┌──────────────────┼──────────────────┐                 │
│         ▼                  ▼                  ▼                 │
│  ┌─────────────┐   ┌─────────────┐   ┌──────────────────┐       │
│  │ SUPERVISADO │   │    NO       │   │  POR REFUERZO    │       │
│  │             │   │ SUPERVISADO │   │  (Reinforcement) │       │
│  └─────────────┘   └─────────────┘   └──────────────────┘       │
│         │                  │                  │                 │
│  "Aprende con    "Encuentra      "Aprende por                   │
│   ejemplos        patrones        prueba y error                │
│   etiquetados"    ocultos"        con recompensas"              │
│                                                                 │
│  Ej: Detectar     Ej: Agrupar     Ej: Jugar ajedrez,            │
│  spam, predecir   clientes,       conducir autónomo             │
│  precios          reducir                                       │
│                   dimensiones                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1. Aprendizaje Supervisado

```
┌─────────────────────────────────────────────────────────────────┐
│               CONCEPTO: APRENDIZAJE SUPERVISADO                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  El modelo aprende de ejemplos ETIQUETADOS (con respuesta).     │
│  Como un estudiante que aprende con exámenes corregidos.        │
│                                                                 │
│  DATOS DE ENTRADA:                                              │
│  ┌────────────────────────────────────────────────────┐         │
│  │  Features (X)              │  Label (y)           │         │
│  │  (Características)         │  (Etiqueta/Respuesta)│         │
│  ├────────────────────────────────────────────────────┤         │
│  │  [área=100m², hab=3]       │  precio=$200,000     │         │
│  │  [área=150m², hab=4]       │  precio=$280,000     │         │
│  │  [área=80m², hab=2]        │  precio=$150,000     │         │
│  └────────────────────────────────────────────────────┘         │
│                                                                 │
│  DOS TIPOS PRINCIPALES:                                         │
│                                                                 │
│  1. REGRESIÓN (predice valores continuos)                       │
│     "¿Cuál será el precio?" → $245,000                          │
│     Algoritmos: Linear Regression, Random Forest, XGBoost       │
│                                                                 │
│  2. CLASIFICACIÓN (predice categorías)                          │
│     "¿Es spam o no spam?" → Spam                                │
│     "¿Qué dígito es?" → 7                                       │
│     Algoritmos: Logistic Regression, SVM, Neural Networks       │
│                                                                 │
│  MÉTRICAS DE ÉXITO:                                             │
│  • Regresión: MSE, RMSE, MAE, R²                                │
│  • Clasificación: Accuracy, Precision, Recall, F1, AUC-ROC      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# EJEMPLO: Regresión - Predecir precios
# ========================================
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score

# Datos: [metros², habitaciones, antigüedad]
X = np.array([
    [100, 3, 10],
    [150, 4, 5],
    [80, 2, 15],
    [120, 3, 8],
    [200, 5, 2]
])
# Etiquetas: precios en miles
y = np.array([200, 280, 150, 220, 350])

# Dividir datos (80% entrenamiento, 20% prueba)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Entrenar modelo
model = LinearRegression()
model.fit(X_train, y_train)

# Predecir
predictions = model.predict(X_test)

# Evaluar
print(f"R² Score: {r2_score(y_test, predictions):.2f}")
print(f"RMSE: {np.sqrt(mean_squared_error(y_test, predictions)):.2f}")

# Predecir casa nueva
new_house = [[110, 3, 7]]
print(f"Precio predicho: ${model.predict(new_house)[0]:.0f}k")
```

```python
# ========================================
# EJEMPLO: Clasificación - Detectar spam
# ========================================
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.metrics import classification_report

# Datos de ejemplo
emails = [
    "Gana dinero gratis ahora",
    "Reunión mañana a las 10",
    "Has ganado $1,000,000",
    "Informe del proyecto adjunto",
    "Oferta especial solo hoy",
    "¿Podemos hablar del presupuesto?"
]
labels = [1, 0, 1, 0, 1, 0]  # 1 = spam, 0 = no spam

# Convertir texto a números (vectorización)
vectorizer = TfidfVectorizer()
X = vectorizer.fit_transform(emails)

# Entrenar clasificador
classifier = MultinomialNB()
classifier.fit(X, labels)

# Predecir nuevo email
new_email = ["Gana premios gratis sin esfuerzo"]
new_X = vectorizer.transform(new_email)
prediction = classifier.predict(new_X)
print(f"¿Es spam?: {'Sí' if prediction[0] == 1 else 'No'}")
```

### 2. Aprendizaje No Supervisado

```
┌─────────────────────────────────────────────────────────────────┐
│             CONCEPTO: APRENDIZAJE NO SUPERVISADO                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  El modelo busca patrones en datos SIN ETIQUETAS.               │
│  Como un niño que agrupa juguetes por color sin que le digan.   │
│                                                                 │
│  DATOS DE ENTRADA:                                              │
│  ┌────────────────────────────────────────────────────┐         │
│  │  Features (X)              │  Label (y)           │         │
│  │  (Características)         │  NO HAY             │         │
│  ├────────────────────────────────────────────────────┤         │
│  │  [edad=25, ingreso=$50k]   │        ?             │         │
│  │  [edad=45, ingreso=$100k]  │        ?             │         │
│  │  [edad=30, ingreso=$60k]   │        ?             │         │
│  └────────────────────────────────────────────────────┘         │
│                                                                 │
│  TIPOS PRINCIPALES:                                             │
│                                                                 │
│  1. CLUSTERING (Agrupación)                                     │
│     Agrupa datos similares                                      │
│     "Estos clientes son parecidos entre sí"                     │
│     Algoritmos: K-Means, DBSCAN, Hierarchical                   │
│                                                                 │
│  2. REDUCCIÓN DE DIMENSIONALIDAD                                │
│     Reduce features manteniendo información                     │
│     "De 100 features a 10 que capturan lo esencial"             │
│     Algoritmos: PCA, t-SNE, UMAP                                │
│                                                                 │
│  3. DETECCIÓN DE ANOMALÍAS                                      │
│     Encuentra puntos "raros"                                    │
│     "Esta transacción no es normal"                             │
│     Algoritmos: Isolation Forest, One-Class SVM                 │
│                                                                 │
│  CASOS DE USO:                                                  │
│  • Segmentación de clientes                                     │
│  • Detección de fraude                                          │
│  • Reducción de datos para visualización                        │
│  • Sistemas de recomendación (collaborative filtering)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# EJEMPLO: Clustering - Segmentar clientes
# ========================================
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

# Datos de clientes: [edad, ingreso_anual, gasto_mensual]
clientes = np.array([
    [25, 30000, 500],
    [30, 35000, 600],
    [45, 80000, 1500],
    [50, 90000, 1800],
    [23, 25000, 400],
    [55, 100000, 2000],
    [28, 32000, 550],
    [48, 85000, 1600]
])

# IMPORTANTE: Normalizar datos para K-Means
scaler = StandardScaler()
clientes_scaled = scaler.fit_transform(clientes)

# Encontrar número óptimo de clusters (Método del codo)
inertias = []
for k in range(1, 6):
    kmeans = KMeans(n_clusters=k, random_state=42)
    kmeans.fit(clientes_scaled)
    inertias.append(kmeans.inertia_)

# Visualizar codo
plt.plot(range(1, 6), inertias, 'bo-')
plt.xlabel('Número de clusters')
plt.ylabel('Inercia')
plt.title('Método del Codo')
plt.show()

# Aplicar K-Means con k=2 (según el codo)
kmeans = KMeans(n_clusters=2, random_state=42)
clusters = kmeans.fit_predict(clientes_scaled)

print("Asignación de clusters:", clusters)
# Resultado: [0, 0, 1, 1, 0, 1, 0, 1]
# Cluster 0: Jóvenes con menor ingreso
# Cluster 1: Mayores con mayor ingreso
```

```python
# ========================================
# EJEMPLO: Reducción de dimensionalidad - PCA
# ========================================
from sklearn.decomposition import PCA
from sklearn.datasets import load_digits
import matplotlib.pyplot as plt

# Cargar dataset de dígitos (64 features por imagen)
digits = load_digits()
X = digits.data  # Shape: (1797, 64)
y = digits.target

print(f"Dimensiones originales: {X.shape}")

# Reducir de 64 a 2 dimensiones para visualizar
pca = PCA(n_components=2)
X_reduced = pca.fit_transform(X)

print(f"Dimensiones reducidas: {X_reduced.shape}")
print(f"Varianza explicada: {sum(pca.explained_variance_ratio_):.2%}")

# Visualizar
plt.figure(figsize=(10, 8))
scatter = plt.scatter(X_reduced[:, 0], X_reduced[:, 1], 
                      c=y, cmap='tab10', alpha=0.6)
plt.colorbar(scatter)
plt.title('Dígitos en 2D (PCA)')
plt.xlabel('Componente Principal 1')
plt.ylabel('Componente Principal 2')
plt.show()
```

### 3. Aprendizaje por Refuerzo

```
┌─────────────────────────────────────────────────────────────────┐
│             CONCEPTO: APRENDIZAJE POR REFUERZO                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  El AGENTE aprende tomando ACCIONES en un AMBIENTE              │
│  y recibiendo RECOMPENSAS o CASTIGOS.                           │
│                                                                 │
│  Como enseñar a un perro: premio por sentarse, nada si no lo    │
│  hace. El perro aprende qué acciones dan premios.               │
│                                                                 │
│  COMPONENTES:                                                   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                                                         │    │
│  │    ┌─────────┐     Acción      ┌───────────┐           │    │
│  │    │  AGENTE │────────────────▶│  AMBIENTE │           │    │
│  │    │         │◀────────────────│           │           │    │
│  │    └─────────┘  Estado +       └───────────┘           │    │
│  │                 Recompensa                              │    │
│  │                                                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  VOCABULARIO:                                                   │
│  • Estado (s): Situación actual del ambiente                    │
│  • Acción (a): Lo que el agente puede hacer                     │
│  • Recompensa (r): Feedback numérico (+1, -1, etc.)             │
│  • Política (π): Estrategia del agente (qué acción en qué estado)│
│  • Valor (V): Recompensa esperada a largo plazo                 │
│                                                                 │
│  ALGORITMOS CLAVE:                                              │
│  • Q-Learning: Aprende valores de estado-acción                 │
│  • Deep Q-Network (DQN): Q-Learning con redes neuronales        │
│  • Policy Gradient: Aprende la política directamente            │
│  • PPO, A3C: Algoritmos modernos de alto rendimiento            │
│                                                                 │
│  CASOS DE USO:                                                  │
│  • Juegos (AlphaGo, videojuegos)                                │
│  • Robótica (caminar, manipular objetos)                        │
│  • Conducción autónoma                                          │
│  • Trading algorítmico                                          │
│  • Optimización de recursos                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# EJEMPLO CONCEPTUAL: Q-Learning simple
# ========================================
import numpy as np

# Ambiente: Laberinto 4x4
# 0 = camino, 1 = muro, 2 = meta
# El agente empieza en (0,0), meta en (3,3)

class SimpleMaze:
    def __init__(self):
        self.size = 4
        self.state = (0, 0)
        self.goal = (3, 3)
        # Acciones: 0=arriba, 1=derecha, 2=abajo, 3=izquierda
        self.actions = [(−1, 0), (0, 1), (1, 0), (0, −1)]
    
    def reset(self):
        self.state = (0, 0)
        return self.state
    
    def step(self, action):
        # Calcular nueva posición
        dx, dy = self.actions[action]
        new_x = max(0, min(self.size−1, self.state[0] + dx))
        new_y = max(0, min(self.size−1, self.state[1] + dy))
        self.state = (new_x, new_y)
        
        # Calcular recompensa
        if self.state == self.goal:
            return self.state, 100, True   # Meta: gran recompensa
        return self.state, −1, False        # Cada paso: pequeño castigo

# Q-Learning
env = SimpleMaze()
q_table = np.zeros((4, 4, 4))  # Estados x Acciones

alpha = 0.1      # Tasa de aprendizaje
gamma = 0.99     # Factor de descuento (importancia del futuro)
epsilon = 0.1    # Exploración vs explotación

for episode in range(1000):
    state = env.reset()
    done = False
    
    while not done:
        # Epsilon-greedy: explora o explota
        if np.random.random() < epsilon:
            action = np.random.randint(4)  # Explorar
        else:
            action = np.argmax(q_table[state])  # Explotar
        
        next_state, reward, done = env.step(action)
        
        # Actualizar Q-table (ecuación de Bellman)
        old_value = q_table[state][action]
        next_max = np.max(q_table[next_state])
        new_value = old_value + alpha * (reward + gamma * next_max − old_value)
        q_table[state][action] = new_value
        
        state = next_state

print("Q-table entrenada. El agente aprendió el camino óptimo.")
```

---

## El Pipeline de Machine Learning

```
┌─────────────────────────────────────────────────────────────────┐
│            PIPELINE COMPLETO DE MACHINE LEARNING                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DEFINIR          2. RECOLECTAR      3. EXPLORAR             │
│  EL PROBLEMA  ──────▶ DATOS      ──────▶ DATOS (EDA)           │
│                                                                 │
│  "¿Qué quiero        "¿De dónde         "¿Cómo son los         │
│   predecir?"          vienen?"           datos?"                │
│                                                                 │
│       │                                                         │
│       ▼                                                         │
│  4. PREPARAR         5. SELECCIONAR     6. ENTRENAR             │
│  DATOS        ──────▶ MODELO     ──────▶ MODELO                │
│                                                                 │
│  "Limpiar,           "¿Qué algoritmo    "Ajustar               │
│   transformar"        usar?"             parámetros"            │
│                                                                 │
│       │                                                         │
│       ▼                                                         │
│  7. EVALUAR          8. OPTIMIZAR       9. DESPLEGAR            │
│  MODELO       ──────▶ (TUNING)   ──────▶ PRODUCCIÓN            │
│                                                                 │
│  "¿Qué tan           "Mejorar           "Poner en              │
│   bueno es?"          hiperparámetros"   uso real"             │
│                                                                 │
│       │                                                         │
│       ▼                                                         │
│  10. MONITOREAR ──────▶ VOLVER A 1 SI ES NECESARIO             │
│  "¿Sigue funcionando bien?"                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Preparación de Datos

```
┌─────────────────────────────────────────────────────────────────┐
│         CONCEPTO: PREPARACIÓN DE DATOS (80% del trabajo)        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  "Garbage in, garbage out" - Si los datos son malos,            │
│  el modelo será malo sin importar el algoritmo.                 │
│                                                                 │
│  PASOS DE PREPARACIÓN:                                          │
│                                                                 │
│  1. LIMPIEZA                                                    │
│     • Valores faltantes (NaN)                                   │
│     • Duplicados                                                │
│     • Outliers (valores extremos)                               │
│     • Errores de formato                                        │
│                                                                 │
│  2. TRANSFORMACIÓN                                              │
│     • Encoding categórico (texto → números)                     │
│     • Scaling/Normalización (misma escala)                      │
│     • Feature engineering (crear nuevas variables)              │
│                                                                 │
│  3. DIVISIÓN                                                    │
│     • Train set (60-80%) - Para entrenar                        │
│     • Validation set (10-20%) - Para ajustar                    │
│     • Test set (10-20%) - Para evaluar final                    │
│                                                                 │
│  ⚠️ IMPORTANTE: NUNCA usar test set para decisiones de modelo   │
│     Solo al final, una vez, para reportar rendimiento real      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# EJEMPLO COMPLETO: Preparación de datos
# ========================================
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, LabelEncoder, OneHotEncoder
from sklearn.impute import SimpleImputer

# Crear dataset de ejemplo
data = pd.DataFrame({
    'edad': [25, 30, np.nan, 45, 50, 35, 40, np.nan, 55, 28],
    'salario': [30000, 45000, 50000, 80000, np.nan, 60000, 70000, 55000, 90000, 35000],
    'ciudad': ['Madrid', 'Barcelona', 'Madrid', 'Valencia', 'Barcelona', 
               'Madrid', 'Valencia', 'Barcelona', 'Madrid', 'Valencia'],
    'experiencia': ['junior', 'mid', 'mid', 'senior', 'senior',
                    'mid', 'senior', 'mid', 'senior', 'junior'],
    'compra': [0, 1, 1, 1, 1, 0, 1, 0, 1, 0]  # Target
})

print("Datos originales:")
print(data)
print(f"\nValores faltantes:\n{data.isnull().sum()}")

# ========================================
# 1. MANEJO DE VALORES FALTANTES
# ========================================

# Opción A: Eliminar filas (si son pocas)
# data_clean = data.dropna()

# Opción B: Imputar con media/mediana/moda
imputer_num = SimpleImputer(strategy='median')  # Para numéricos
data['edad'] = imputer_num.fit_transform(data[['edad']])
data['salario'] = imputer_num.fit_transform(data[['salario']])

print("\nDespués de imputar:")
print(data)

# ========================================
# 2. ENCODING DE VARIABLES CATEGÓRICAS
# ========================================

# Opción A: Label Encoding (para ordinales: junior < mid < senior)
le = LabelEncoder()
data['experiencia_encoded'] = le.fit_transform(data['experiencia'])
# junior=0, mid=1, senior=2

# Opción B: One-Hot Encoding (para nominales sin orden: ciudades)
data = pd.get_dummies(data, columns=['ciudad'], prefix='ciudad')

print("\nDespués de encoding:")
print(data.columns.tolist())

# ========================================
# 3. ESCALADO DE FEATURES
# ========================================

# Separar features y target
X = data.drop(['compra', 'experiencia'], axis=1)
y = data['compra']

# StandardScaler: Media=0, Std=1 (mejor para la mayoría)
# MinMaxScaler: Rango [0,1] (mejor para redes neuronales)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X.select_dtypes(include=[np.number]))

print(f"\nMedia después de escalar: {X_scaled.mean(axis=0)}")
print(f"Std después de escalar: {X_scaled.std(axis=0)}")

# ========================================
# 4. DIVISIÓN DE DATOS
# ========================================

X_train, X_temp, y_train, y_temp = train_test_split(
    X_scaled, y, test_size=0.3, random_state=42, stratify=y
)
X_val, X_test, y_val, y_test = train_test_split(
    X_temp, y_temp, test_size=0.5, random_state=42
)

print(f"\nTamaños:")
print(f"  Train: {len(X_train)} ({len(X_train)/len(X_scaled)*100:.0f}%)")
print(f"  Val:   {len(X_val)} ({len(X_val)/len(X_scaled)*100:.0f}%)")
print(f"  Test:  {len(X_test)} ({len(X_test)/len(X_scaled)*100:.0f}%)")
```

### Feature Engineering

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: FEATURE ENGINEERING                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Es el ARTE de crear nuevas variables a partir de las           │
│  existentes para que el modelo aprenda mejor.                   │
│                                                                 │
│  "Un buen feature vale más que un algoritmo complejo"           │
│                                                                 │
│  TÉCNICAS COMUNES:                                              │
│                                                                 │
│  1. COMBINACIONES MATEMÁTICAS                                   │
│     • ratio_precio_area = precio / area                         │
│     • edad_al_cuadrado = edad ** 2                              │
│     • log_salario = np.log(salario)                             │
│                                                                 │
│  2. EXTRACCIÓN DE FECHAS                                        │
│     • dia_semana, mes, año, trimestre                           │
│     • es_fin_de_semana, es_festivo                              │
│     • dias_desde_ultimo_evento                                  │
│                                                                 │
│  3. AGREGACIONES                                                │
│     • promedio_compras_por_cliente                              │
│     • total_transacciones_ultimo_mes                            │
│                                                                 │
│  4. BINNING (Discretización)                                    │
│     • edad → ['joven', 'adulto', 'mayor']                       │
│                                                                 │
│  5. TEXT FEATURES                                               │
│     • longitud_texto, num_palabras                              │
│     • contiene_palabra_clave                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# EJEMPLOS DE FEATURE ENGINEERING
# ========================================
import pandas as pd
import numpy as np

# Dataset de casas
houses = pd.DataFrame({
    'precio': [200000, 350000, 150000, 500000],
    'area': [100, 150, 80, 200],
    'habitaciones': [3, 4, 2, 5],
    'fecha_construccion': ['2010-05-15', '2020-01-20', '1995-08-10', '2022-03-01'],
    'descripcion': ['Casa bonita cerca del centro', 
                    'Amplia casa con jardín grande y piscina', 
                    'Apartamento pequeño', 
                    'Lujosa mansión con 5 habitaciones y gimnasio']
})

# 1. Combinaciones matemáticas
houses['precio_por_m2'] = houses['precio'] / houses['area']
houses['habitaciones_por_m2'] = houses['habitaciones'] / houses['area']
houses['log_precio'] = np.log(houses['precio'])

# 2. Features de fechas
houses['fecha_construccion'] = pd.to_datetime(houses['fecha_construccion'])
houses['año_construccion'] = houses['fecha_construccion'].dt.year
houses['antiguedad'] = 2026 - houses['año_construccion']
houses['decada'] = (houses['año_construccion'] // 10) * 10

# 3. Binning
houses['categoria_precio'] = pd.cut(
    houses['precio'],
    bins=[0, 200000, 400000, np.inf],
    labels=['económico', 'medio', 'premium']
)

# 4. Text features
houses['longitud_descripcion'] = houses['descripcion'].str.len()
houses['num_palabras'] = houses['descripcion'].str.split().str.len()
houses['tiene_piscina'] = houses['descripcion'].str.contains('piscina').astype(int)
houses['tiene_jardin'] = houses['descripcion'].str.contains('jardín').astype(int)

print(houses[['precio', 'precio_por_m2', 'antiguedad', 'categoria_precio', 
              'longitud_descripcion', 'tiene_piscina']].to_string())
```

---

## Métricas y Evaluación

### Métricas de Regresión

```
┌─────────────────────────────────────────────────────────────────┐
│                 MÉTRICAS DE REGRESIÓN                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Todas miden "qué tan lejos están las predicciones de la        │
│  realidad". Menos es mejor (excepto R²).                        │
│                                                                 │
│  1. MAE (Mean Absolute Error)                                   │
│     Promedio de |error|                                         │
│     MAE = Σ|y_real - y_pred| / n                                │
│     ✅ Fácil de interpretar ("error promedio de $X")            │
│     ✅ Robusto a outliers                                       │
│                                                                 │
│  2. MSE (Mean Squared Error)                                    │
│     Promedio de error²                                          │
│     MSE = Σ(y_real - y_pred)² / n                               │
│     ✅ Penaliza errores grandes más                             │
│     ❌ Unidades al cuadrado (difícil interpretar)               │
│                                                                 │
│  3. RMSE (Root Mean Squared Error)                              │
│     Raíz de MSE                                                 │
│     RMSE = √MSE                                                 │
│     ✅ Mismas unidades que y                                    │
│     ✅ Más usado en práctica                                    │
│                                                                 │
│  4. R² (Coeficiente de determinación)                           │
│     "Qué % de varianza explica el modelo"                       │
│     R² = 1 - (SS_res / SS_tot)                                  │
│     ✅ Entre 0 y 1 (1 = perfecto)                               │
│     ❌ Puede ser negativo si modelo muy malo                    │
│                                                                 │
│  ¿CUÁL USAR?                                                    │
│  • Comparar modelos: R², RMSE                                   │
│  • Comunicar a negocio: MAE ("nos equivocamos $500 promedio")  │
│  • Datos con outliers: MAE                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# MÉTRICAS DE REGRESIÓN
# ========================================
from sklearn.metrics import (
    mean_absolute_error, 
    mean_squared_error, 
    r2_score,
    mean_absolute_percentage_error
)
import numpy as np

y_real = np.array([100, 150, 200, 250, 300])
y_pred = np.array([110, 145, 190, 260, 310])

mae = mean_absolute_error(y_real, y_pred)
mse = mean_squared_error(y_real, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_real, y_pred)
mape = mean_absolute_percentage_error(y_real, y_pred)

print(f"MAE:  {mae:.2f}  (error promedio absoluto)")
print(f"MSE:  {mse:.2f}  (error cuadrático medio)")
print(f"RMSE: {rmse:.2f} (raíz del MSE, mismas unidades)")
print(f"R²:   {r2:.3f}   (1.0 = perfecto)")
print(f"MAPE: {mape:.2%} (error porcentual)")
```

### Métricas de Clasificación

```
┌─────────────────────────────────────────────────────────────────┐
│                MÉTRICAS DE CLASIFICACIÓN                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MATRIZ DE CONFUSIÓN:                                           │
│                                                                 │
│                         PREDICHO                                │
│                    Positivo  │  Negativo                        │
│               ┌─────────────┼─────────────┐                     │
│  REAL    Pos  │     TP      │     FN      │                     │
│               │  (Acierto)  │ (Error II)  │                     │
│               ├─────────────┼─────────────┤                     │
│          Neg  │     FP      │     TN      │                     │
│               │ (Error I)   │  (Acierto)  │                     │
│               └─────────────┴─────────────┘                     │
│                                                                 │
│  • TP (True Positive): Era spam, predije spam ✓                 │
│  • TN (True Negative): No era spam, predije no spam ✓           │
│  • FP (False Positive): No era spam, predije spam ✗             │
│  • FN (False Negative): Era spam, predije no spam ✗             │
│                                                                 │
│  MÉTRICAS DERIVADAS:                                            │
│                                                                 │
│  ACCURACY = (TP + TN) / Total                                   │
│  "De todos, ¿cuántos acerté?"                                   │
│  ⚠️ ENGAÑOSA con clases desbalanceadas                          │
│                                                                 │
│  PRECISION = TP / (TP + FP)                                     │
│  "De los que dije positivo, ¿cuántos eran realmente?"           │
│  Importante cuando FP es costoso (ej: acusar inocente)          │
│                                                                 │
│  RECALL (Sensitivity) = TP / (TP + FN)                          │
│  "De los realmente positivos, ¿cuántos encontré?"               │
│  Importante cuando FN es costoso (ej: no detectar cáncer)       │
│                                                                 │
│  F1-SCORE = 2 × (Precision × Recall) / (Precision + Recall)     │
│  Balance entre precision y recall                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│         ¿QUÉ MÉTRICA USAR EN CLASIFICACIÓN?                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CASO                              │  MÉTRICA CLAVE             │
│  ─────────────────────────────────────────────────────────────  │
│  Detección de fraude               │  Recall (no perder fraude) │
│  Diagnóstico médico                │  Recall (no perder casos)  │
│  Filtro de spam                    │  Precision (no perder mail)│
│  Clases balanceadas, general       │  F1-Score o Accuracy       │
│  Necesito ranking/probabilidades   │  AUC-ROC                   │
│  Clases muy desbalanceadas         │  AUC-PR (Precision-Recall) │
│                                                                 │
│  REGLA GENERAL:                                                 │
│  • ¿Qué error es peor?                                          │
│  • Si FN es peor → Maximiza Recall                              │
│  • Si FP es peor → Maximiza Precision                           │
│  • Ambos igual de malos → Maximiza F1                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# MÉTRICAS DE CLASIFICACIÓN
# ========================================
from sklearn.metrics import (
    confusion_matrix,
    accuracy_score,
    precision_score,
    recall_score,
    f1_score,
    classification_report,
    roc_auc_score,
    roc_curve
)
import matplotlib.pyplot as plt

y_real = [1, 1, 1, 1, 0, 0, 0, 0, 1, 0]
y_pred = [1, 1, 0, 1, 0, 1, 0, 0, 1, 0]
y_prob = [0.9, 0.8, 0.4, 0.7, 0.2, 0.6, 0.3, 0.1, 0.85, 0.15]

# Matriz de confusión
cm = confusion_matrix(y_real, y_pred)
print("Matriz de Confusión:")
print(cm)
# [[3 1]    TN=3, FP=1
#  [1 5]]   FN=1, TP=4

# Métricas
print(f"\nAccuracy:  {accuracy_score(y_real, y_pred):.2f}")
print(f"Precision: {precision_score(y_real, y_pred):.2f}")
print(f"Recall:    {recall_score(y_real, y_pred):.2f}")
print(f"F1-Score:  {f1_score(y_real, y_pred):.2f}")
print(f"AUC-ROC:   {roc_auc_score(y_real, y_prob):.2f}")

# Reporte completo
print("\nReporte Completo:")
print(classification_report(y_real, y_pred, target_names=['No Spam', 'Spam']))

# Curva ROC
fpr, tpr, thresholds = roc_curve(y_real, y_prob)
plt.figure(figsize=(8, 6))
plt.plot(fpr, tpr, label=f'AUC = {roc_auc_score(y_real, y_prob):.2f}')
plt.plot([0, 1], [0, 1], 'k--', label='Random')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Curva ROC')
plt.legend()
plt.show()
```

---

## Overfitting y Underfitting

```
┌─────────────────────────────────────────────────────────────────┐
│           CONCEPTO: OVERFITTING vs UNDERFITTING                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  El balance más importante en ML: Complejidad del modelo        │
│                                                                 │
│                                                                 │
│  UNDERFITTING          GOOD FIT           OVERFITTING           │
│  (Subajuste)           (Buen ajuste)      (Sobreajuste)         │
│                                                                 │
│     ───────              ~~~                ~∿~∿~∿~             │
│    /       \            /   \              /\/\/\/\             │
│   •    •    •          •     •            •       •             │
│  •          •         •       •          •         •            │
│                                                                 │
│  "Modelo muy           "Captura el        "Memoriza el          │
│   simple, no            patrón real"       ruido"               │
│   captura patrón"                                               │
│                                                                 │
│  Señales:              Señales:           Señales:              │
│  • Train error alto    • Train error      • Train error bajo    │
│  • Test error alto       bajo             • Test error alto     │
│                        • Test error       • Gran diferencia     │
│                          bajo               entre ambos         │
│                                                                 │
│  BIAS-VARIANCE TRADEOFF:                                        │
│                                                                 │
│  • BIAS (Sesgo): Error por suposiciones simplificadoras         │
│    Alto bias = Underfitting                                     │
│                                                                 │
│  • VARIANCE (Varianza): Sensibilidad a datos de entrenamiento   │
│    Alta varianza = Overfitting                                  │
│                                                                 │
│  Total Error = Bias² + Variance + Ruido irreducible             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│           CÓMO DETECTAR Y SOLUCIONAR                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  UNDERFITTING (Alto Bias):                                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  SÍNTOMAS:                                               │   │
│  │  • Rendimiento pobre en train Y test                     │   │
│  │  • Learning curve: ambas métricas altas y cercanas       │   │
│  │                                                          │   │
│  │  SOLUCIONES:                                             │   │
│  │  • Usar modelo más complejo                              │   │
│  │  • Añadir más features                                   │   │
│  │  • Reducir regularización                                │   │
│  │  • Más tiempo de entrenamiento                           │   │
│  │  • Feature engineering                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  OVERFITTING (Alta Varianza):                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  SÍNTOMAS:                                               │   │
│  │  • Train: excelente, Test: pobre                         │   │
│  │  • Learning curve: train baja, test alta (gap grande)    │   │
│  │                                                          │   │
│  │  SOLUCIONES:                                             │   │
│  │  • Más datos de entrenamiento                            │   │
│  │  • Regularización (L1, L2, Dropout)                      │   │
│  │  • Modelo más simple                                     │   │
│  │  • Early stopping                                        │   │
│  │  • Cross-validation                                      │   │
│  │  • Data augmentation                                     │   │
│  │  • Reducir features (selección)                          │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# EJEMPLO: Detectar Overfitting con Learning Curves
# ========================================
import numpy as np
import matplotlib.pyplot as plt
from sklearn.model_selection import learning_curve
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import make_classification

# Crear dataset
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)

# Modelo que puede hacer overfitting fácilmente
model = DecisionTreeClassifier(max_depth=None)  # Sin límite de profundidad

# Calcular learning curves
train_sizes, train_scores, test_scores = learning_curve(
    model, X, y,
    train_sizes=np.linspace(0.1, 1.0, 10),
    cv=5,
    scoring='accuracy'
)

# Promedios y desviaciones
train_mean = train_scores.mean(axis=1)
train_std = train_scores.std(axis=1)
test_mean = test_scores.mean(axis=1)
test_std = test_scores.std(axis=1)

# Visualizar
plt.figure(figsize=(10, 6))
plt.plot(train_sizes, train_mean, label='Train score', color='blue')
plt.fill_between(train_sizes, train_mean - train_std, train_mean + train_std, 
                 alpha=0.1, color='blue')
plt.plot(train_sizes, test_mean, label='Test score', color='red')
plt.fill_between(train_sizes, test_mean - test_std, test_mean + test_std, 
                 alpha=0.1, color='red')

plt.xlabel('Training Set Size')
plt.ylabel('Accuracy')
plt.title('Learning Curves - Detectar Overfitting')
plt.legend()
plt.grid(True)
plt.show()

# Si hay gap grande entre train (alto) y test (bajo) → Overfitting
print(f"Train score final: {train_mean[-1]:.3f}")
print(f"Test score final:  {test_mean[-1]:.3f}")
print(f"Gap: {train_mean[-1] - test_mean[-1]:.3f}")
```

### Técnicas de Regularización

```python
# ========================================
# REGULARIZACIÓN: Prevenir Overfitting
# ========================================
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier

# 1. REGULARIZACIÓN L2 (Ridge)
# Penaliza coeficientes grandes, los reduce pero no los elimina
ridge = Ridge(alpha=1.0)  # alpha = fuerza de regularización

# 2. REGULARIZACIÓN L1 (Lasso)
# Puede llevar coeficientes a exactamente 0 (selección de features)
lasso = Lasso(alpha=0.1)

# 3. ELASTIC NET (Combinación L1 + L2)
elastic = ElasticNet(alpha=0.1, l1_ratio=0.5)

# 4. PARA ÁRBOLES: Limitar complejidad
tree = DecisionTreeClassifier(
    max_depth=5,           # Profundidad máxima
    min_samples_split=10,  # Mínimo samples para dividir
    min_samples_leaf=5,    # Mínimo samples en hoja
    max_features='sqrt'    # Número de features a considerar
)

# 5. PARA RANDOM FOREST: Menos árboles, más restricciones
rf = RandomForestClassifier(
    n_estimators=100,      # Número de árboles
    max_depth=10,          # Limitar profundidad
    min_samples_leaf=5,
    max_features='sqrt',
    random_state=42
)
```

### Cross-Validation

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: CROSS-VALIDATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Técnica para evaluar modelos de forma más robusta.             │
│  En vez de una sola división train/test, hace múltiples.        │
│                                                                 │
│  K-FOLD CROSS-VALIDATION (K=5):                                 │
│                                                                 │
│  Datos: [████████████████████████████████████████]              │
│                                                                 │
│  Fold 1: [TEST][train][train][train][train]  → Score 1          │
│  Fold 2: [train][TEST][train][train][train]  → Score 2          │
│  Fold 3: [train][train][TEST][train][train]  → Score 3          │
│  Fold 4: [train][train][train][TEST][train]  → Score 4          │
│  Fold 5: [train][train][train][train][TEST]  → Score 5          │
│                                                                 │
│  Score Final = Promedio(Score 1-5) ± Desviación                 │
│                                                                 │
│  VENTAJAS:                                                      │
│  • Cada dato es usado para test exactamente una vez             │
│  • Estimación más robusta del rendimiento real                  │
│  • Detecta si el modelo es inestable                            │
│                                                                 │
│  VARIANTES:                                                     │
│  • Stratified K-Fold: Mantiene proporción de clases             │
│  • Leave-One-Out: K = número de muestras (muy lento)            │
│  • Time Series Split: Para datos temporales (orden importa)     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# CROSS-VALIDATION EN PRÁCTICA
# ========================================
from sklearn.model_selection import (
    cross_val_score, 
    StratifiedKFold,
    cross_validate
)
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification

# Crear datos
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)

# Cross-validation simple
scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')
print(f"Accuracy: {scores.mean():.3f} ± {scores.std():.3f}")

# Cross-validation con múltiples métricas
results = cross_validate(
    model, X, y, cv=5,
    scoring=['accuracy', 'precision', 'recall', 'f1'],
    return_train_score=True
)

print("\nResultados detallados:")
for metric in ['accuracy', 'precision', 'recall', 'f1']:
    train = results[f'train_{metric}']
    test = results[f'test_{metric}']
    print(f"{metric}:")
    print(f"  Train: {train.mean():.3f} ± {train.std():.3f}")
    print(f"  Test:  {test.mean():.3f} ± {test.std():.3f}")

# Stratified K-Fold (mantiene proporciones de clases)
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=skf, scoring='accuracy')
print(f"\nStratified K-Fold: {scores.mean():.3f} ± {scores.std():.3f}")
```

---

## 🏷️ Tags

#programming #machinelearning #python #datascience #ai

---

## 📚 Ver También

- [[ML Algorithms Guide|Algoritmos de ML - Guía Detallada]]
- [[Deep Learning Guide|Deep Learning - Guía Completa]]
- [[ML Projects Guide|Proyectos Prácticos de ML]]
