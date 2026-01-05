# Machine Learning - Algoritmos Principales

> **Guía de los algoritmos más importantes con explicaciones conceptuales**
> Última actualización: Enero 2026

---

## Tabla de Contenidos

1. [[#Algoritmos de Regresión|Regresión]]
2. [[#Algoritmos de Clasificación|Clasificación]]
3. [[#Árboles de Decisión|Árboles de Decisión]]
4. [[#Ensemble Methods|Métodos Ensemble]]
5. [[#Support Vector Machines|SVM]]
6. [[#K-Nearest Neighbors|KNN]]

---

## Algoritmos de Regresión

### Linear Regression (Regresión Lineal)

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: REGRESIÓN LINEAL                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  El algoritmo más simple: encuentra una línea recta que         │
│  mejor se ajusta a los datos.                                   │
│                                                                 │
│  ECUACIÓN:  y = β₀ + β₁x₁ + β₂x₂ + ... + βₙxₙ                  │
│                                                                 │
│  • y = valor predicho                                           │
│  • β₀ = intercepto (valor cuando todas las x son 0)             │
│  • βᵢ = coeficientes (cuánto afecta cada feature)               │
│  • xᵢ = features (características)                              │
│                                                                 │
│  VISUALIZACIÓN (1 feature):                                     │
│                                                                 │
│    y │         •    /                                           │
│      │       •   /                                              │
│      │     •  /   ← Línea que minimiza errores                  │
│      │   • /                                                    │
│      │ • /                                                      │
│      └──────────── x                                            │
│                                                                 │
│  ¿CÓMO ENCUENTRA LA MEJOR LÍNEA?                                │
│  Minimiza la suma de errores al cuadrado (OLS):                 │
│  min Σ(y_real - y_pred)²                                        │
│                                                                 │
│  SUPUESTOS (para que funcione bien):                            │
│  1. Relación lineal entre X e y                                 │
│  2. Errores normalmente distribuidos                            │
│  3. No multicolinealidad (features no correlacionadas)          │
│  4. Homocedasticidad (varianza constante de errores)            │
│                                                                 │
│  CUÁNDO USAR:                                                   │
│  ✅ Relación aproximadamente lineal                             │
│  ✅ Necesitas interpretabilidad                                 │
│  ✅ Baseline rápido                                             │
│  ❌ Relaciones complejas/no lineales                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# REGRESIÓN LINEAL COMPLETA
# ========================================
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score

# Datos de ejemplo: Predecir salario por años de experiencia
np.random.seed(42)
experiencia = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]).reshape(-1, 1)
salario = np.array([30, 35, 40, 48, 55, 60, 68, 75, 82, 90]) + np.random.randn(10) * 3

# Dividir datos
X_train, X_test, y_train, y_test = train_test_split(
    experiencia, salario, test_size=0.2, random_state=42
)

# Entrenar modelo
model = LinearRegression()
model.fit(X_train, y_train)

# Coeficientes (interpretables)
print(f"Intercepto (β₀): {model.intercept_:.2f}")
print(f"Coeficiente (β₁): {model.coef_[0]:.2f}")
print(f"\nInterpretación: Por cada año de experiencia,")
print(f"el salario aumenta ${model.coef_[0]:.2f}k")

# Predecir
y_pred = model.predict(X_test)
print(f"\nR² Score: {r2_score(y_test, y_pred):.3f}")
print(f"RMSE: {np.sqrt(mean_squared_error(y_test, y_pred)):.2f}k")

# Visualizar
plt.figure(figsize=(10, 6))
plt.scatter(experiencia, salario, color='blue', label='Datos reales')
plt.plot(experiencia, model.predict(experiencia), color='red', label='Predicción')
plt.xlabel('Años de Experiencia')
plt.ylabel('Salario (miles)')
plt.title('Regresión Lineal: Salario vs Experiencia')
plt.legend()
plt.grid(True)
plt.show()
```

### Polynomial Regression

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: REGRESIÓN POLINOMIAL                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Extensión de lineal para capturar relaciones curvas.           │
│  Añade potencias de las features como nuevas features.          │
│                                                                 │
│  Lineal:     y = β₀ + β₁x                                       │
│  Grado 2:    y = β₀ + β₁x + β₂x²                                │
│  Grado 3:    y = β₀ + β₁x + β₂x² + β₃x³                         │
│                                                                 │
│  VISUALIZACIÓN:                                                 │
│                                                                 │
│  Lineal         Grado 2          Grado 3                        │
│    /              ∩                ∿                            │
│   /              / \              /\                            │
│  /              /   \            /  \                           │
│                                     \/                          │
│                                                                 │
│  ⚠️ CUIDADO: Grado muy alto = Overfitting                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import Pipeline

# Datos con relación cuadrática
X = np.array([1, 2, 3, 4, 5, 6, 7, 8]).reshape(-1, 1)
y = np.array([1, 4, 9, 16, 25, 36, 49, 64]) + np.random.randn(8) * 2

# Pipeline: transformar + modelo
poly_model = Pipeline([
    ('poly', PolynomialFeatures(degree=2)),  # Añade x²
    ('linear', LinearRegression())
])

poly_model.fit(X, y)
y_pred = poly_model.predict(X)

print(f"R²: {r2_score(y, y_pred):.3f}")

# Comparar diferentes grados
plt.figure(figsize=(12, 4))
for i, degree in enumerate([1, 2, 5], 1):
    plt.subplot(1, 3, i)
    model = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('linear', LinearRegression())
    ])
    model.fit(X, y)
    
    X_plot = np.linspace(0, 9, 100).reshape(-1, 1)
    plt.scatter(X, y, color='blue')
    plt.plot(X_plot, model.predict(X_plot), color='red')
    plt.title(f'Grado {degree}')
plt.tight_layout()
plt.show()
```

### Ridge y Lasso (Regularización)

```
┌─────────────────────────────────────────────────────────────────┐
│           CONCEPTO: REGULARIZACIÓN L1 y L2                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Añaden una "penalización" para evitar coeficientes muy grandes │
│  y prevenir overfitting.                                        │
│                                                                 │
│  RIDGE (L2): Penaliza suma de coeficientes al cuadrado          │
│  Loss = MSE + α × Σβ²                                           │
│  → Reduce coeficientes pero no los elimina                      │
│  → Bueno cuando todas las features son útiles                   │
│                                                                 │
│  LASSO (L1): Penaliza suma de valores absolutos                 │
│  Loss = MSE + α × Σ|β|                                          │
│  → Puede hacer coeficientes EXACTAMENTE 0                       │
│  → Selección automática de features                             │
│  → Bueno para eliminar features irrelevantes                    │
│                                                                 │
│  ELASTIC NET: Combinación de ambos                              │
│  Loss = MSE + α₁ × Σ|β| + α₂ × Σβ²                              │
│                                                                 │
│  α (alpha) = Fuerza de regularización                           │
│  • α = 0: Sin regularización (igual que Linear)                 │
│  • α grande: Mucha regularización (coeficientes → 0)            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.preprocessing import StandardScaler

# Datos con muchas features (algunas irrelevantes)
np.random.seed(42)
n_samples = 100
X = np.random.randn(n_samples, 10)  # 10 features
# Solo las primeras 3 features son relevantes
y = 3*X[:, 0] + 2*X[:, 1] - X[:, 2] + np.random.randn(n_samples) * 0.5

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Escalar es importante para regularización
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# Comparar modelos
models = {
    'Linear': LinearRegression(),
    'Ridge (L2)': Ridge(alpha=1.0),
    'Lasso (L1)': Lasso(alpha=0.1),
    'ElasticNet': ElasticNet(alpha=0.1, l1_ratio=0.5)
}

print("Comparación de coeficientes:\n")
print(f"{'Modelo':<15} {'R²':<8} Coeficientes")
print("-" * 70)

for name, model in models.items():
    model.fit(X_train_scaled, y_train)
    score = model.score(X_test_scaled, y_test)
    coefs = model.coef_
    print(f"{name:<15} {score:.3f}   {np.round(coefs, 2)}")

# Lasso hace algunos coeficientes = 0 (features 4-10 son irrelevantes)
```

---

## Algoritmos de Clasificación

### Logistic Regression

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: REGRESIÓN LOGÍSTICA                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  A pesar del nombre, es para CLASIFICACIÓN, no regresión.       │
│  Predice la PROBABILIDAD de pertenecer a una clase.             │
│                                                                 │
│  FUNCIÓN SIGMOIDE:                                              │
│                                                                 │
│     P(y=1) = 1 / (1 + e^(-z))    donde z = β₀ + β₁x₁ + ...     │
│                                                                 │
│       1 ┤         ___________                                   │
│         │        /                                              │
│     0.5 ┤───────/────────────  ← Umbral de decisión             │
│         │      /                                                │
│       0 ┤_____/                                                 │
│         └─────────────────────                                  │
│              -4  -2   0   2   4                                 │
│                                                                 │
│  Si P(y=1) > 0.5 → Predice clase 1                              │
│  Si P(y=1) ≤ 0.5 → Predice clase 0                              │
│                                                                 │
│  VENTAJAS:                                                      │
│  ✅ Probabilidades interpretables                               │
│  ✅ Coeficientes interpretables                                 │
│  ✅ Rápido de entrenar                                          │
│  ✅ Funciona bien con features linealmente separables           │
│                                                                 │
│  LIMITACIONES:                                                  │
│  ❌ Asume relación lineal entre features y log-odds             │
│  ❌ No captura relaciones complejas                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_classification
from sklearn.metrics import classification_report, roc_auc_score

# Crear datos de clasificación binaria
X, y = make_classification(
    n_samples=1000, n_features=10, n_informative=5,
    n_redundant=2, random_state=42
)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Entrenar
model = LogisticRegression(random_state=42)
model.fit(X_train, y_train)

# Predecir clases
y_pred = model.predict(X_test)

# Predecir PROBABILIDADES (muy útil)
y_prob = model.predict_proba(X_test)[:, 1]

print("Reporte de Clasificación:")
print(classification_report(y_test, y_pred))
print(f"AUC-ROC: {roc_auc_score(y_test, y_prob):.3f}")

# Interpretar coeficientes
print("\nImportancia de features (coeficientes):")
for i, coef in enumerate(model.coef_[0]):
    print(f"  Feature {i}: {coef:+.3f}")
```

### Naive Bayes

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: NAIVE BAYES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Basado en el Teorema de Bayes con la suposición "ingenua"      │
│  de que todas las features son independientes.                  │
│                                                                 │
│  TEOREMA DE BAYES:                                              │
│                                                                 │
│  P(clase|features) = P(features|clase) × P(clase)               │
│                      ─────────────────────────────              │
│                            P(features)                          │
│                                                                 │
│  "¿Cuál es la probabilidad de que sea spam dado que             │
│   contiene 'gratis' y 'dinero'?"                                │
│                                                                 │
│  TIPOS:                                                         │
│  • GaussianNB: Features continuas (distribución normal)         │
│  • MultinomialNB: Features discretas (conteo de palabras)       │
│  • BernoulliNB: Features binarias (0/1)                         │
│                                                                 │
│  VENTAJAS:                                                      │
│  ✅ MUY rápido (ideal para textos)                              │
│  ✅ Funciona bien con pocos datos                               │
│  ✅ Escala bien a muchas features                               │
│  ✅ Probabilidades calibradas                                   │
│                                                                 │
│  LIMITACIONES:                                                  │
│  ❌ Asume independencia (raramente verdad)                      │
│  ❌ No captura interacciones entre features                     │
│                                                                 │
│  CASO DE USO IDEAL: Clasificación de texto (spam, sentimiento)  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.naive_bayes import GaussianNB, MultinomialNB
from sklearn.feature_extraction.text import CountVectorizer

# ========================================
# EJEMPLO 1: Clasificación de texto (spam)
# ========================================
textos = [
    "oferta gratis dinero ahora",
    "reunión mañana 10am",
    "gana premio millonario",
    "informe proyecto adjunto",
    "descuento especial urgente",
    "confirmación cita médico"
]
labels = [1, 0, 1, 0, 1, 0]  # 1=spam, 0=no spam

# Vectorizar texto (convertir a números)
vectorizer = CountVectorizer()
X = vectorizer.fit_transform(textos)

# Entrenar Naive Bayes Multinomial (para conteos)
nb = MultinomialNB()
nb.fit(X, labels)

# Predecir nuevo email
nuevo = ["dinero gratis sin esfuerzo"]
nuevo_vec = vectorizer.transform(nuevo)
prob = nb.predict_proba(nuevo_vec)[0]
print(f"P(no spam): {prob[0]:.2%}")
print(f"P(spam): {prob[1]:.2%}")

# ========================================
# EJEMPLO 2: Features numéricas
# ========================================
from sklearn.datasets import load_iris

iris = load_iris()
X_train, X_test, y_train, y_test = train_test_split(
    iris.data, iris.target, test_size=0.2
)

gnb = GaussianNB()
gnb.fit(X_train, y_train)
print(f"\nIris Accuracy: {gnb.score(X_test, y_test):.3f}")
```

---

## Árboles de Decisión

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: ÁRBOLES DE DECISIÓN                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Aprende reglas de decisión en forma de árbol.                  │
│  Como un juego de "20 preguntas".                               │
│                                                                 │
│  ESTRUCTURA:                                                    │
│                                                                 │
│             [¿Edad > 30?]           ← Nodo raíz                 │
│              /        \                                         │
│           Sí          No                                        │
│           /            \                                        │
│   [¿Ingreso > 50k?]    [¿Estudiante?]  ← Nodos internos         │
│      /      \            /      \                               │
│    Sí       No         Sí       No                              │
│    /         \         /          \                             │
│  [Compra]  [No]    [Compra]    [No]    ← Hojas (decisión)       │
│                                                                 │
│  ¿CÓMO DECIDE QUÉ PREGUNTA HACER?                               │
│  Busca la división que mejor separa las clases usando:          │
│  • Gini Impurity: Mide "mezcla" de clases (0=puro)              │
│  • Entropy/Information Gain: Mide desorden                      │
│                                                                 │
│  VENTAJAS:                                                      │
│  ✅ MUY interpretable (puedes ver las reglas)                   │
│  ✅ No requiere escalado de datos                               │
│  ✅ Maneja features categóricas y numéricas                     │
│  ✅ Captura relaciones no lineales                              │
│                                                                 │
│  LIMITACIONES:                                                  │
│  ❌ Propenso a overfitting                                      │
│  ❌ Inestable (pequeños cambios → árbol diferente)              │
│  ❌ Sesgado hacia features con muchos niveles                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.tree import DecisionTreeClassifier, plot_tree
from sklearn.datasets import load_iris
import matplotlib.pyplot as plt

# Cargar datos
iris = load_iris()
X, y = iris.data, iris.target

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Entrenar árbol (con límites para evitar overfitting)
tree = DecisionTreeClassifier(
    max_depth=3,           # Profundidad máxima
    min_samples_split=10,  # Mínimo para dividir
    min_samples_leaf=5,    # Mínimo en hojas
    random_state=42
)
tree.fit(X_train, y_train)

print(f"Accuracy: {tree.score(X_test, y_test):.3f}")

# Visualizar el árbol
plt.figure(figsize=(20, 10))
plot_tree(
    tree,
    feature_names=iris.feature_names,
    class_names=iris.target_names,
    filled=True,
    rounded=True,
    fontsize=10
)
plt.title("Árbol de Decisión - Iris")
plt.tight_layout()
plt.show()

# Ver importancia de features
print("\nImportancia de features:")
for name, imp in zip(iris.feature_names, tree.feature_importances_):
    print(f"  {name}: {imp:.3f}")
```

---

## Ensemble Methods

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: MÉTODOS ENSEMBLE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  "La sabiduría de las multitudes"                               │
│  Combinar múltiples modelos débiles → modelo fuerte             │
│                                                                 │
│  DOS ESTRATEGIAS PRINCIPALES:                                   │
│                                                                 │
│  1. BAGGING (Bootstrap Aggregating)                             │
│     • Entrena modelos en PARALELO                               │
│     • Cada modelo ve una muestra diferente (con reemplazo)      │
│     • Combina por votación (clasificación) o promedio (reg)     │
│     • Reduce VARIANZA (overfitting)                             │
│     Ejemplo: Random Forest                                      │
│                                                                 │
│     Datos ─┬─▶ Modelo 1 ──┐                                     │
│            ├─▶ Modelo 2 ──┼──▶ Votación ──▶ Predicción          │
│            └─▶ Modelo 3 ──┘                                     │
│                                                                 │
│  2. BOOSTING                                                    │
│     • Entrena modelos en SECUENCIA                              │
│     • Cada modelo corrige errores del anterior                  │
│     • Reduce BIAS (underfitting)                                │
│     Ejemplos: AdaBoost, Gradient Boosting, XGBoost              │
│                                                                 │
│     Datos ─▶ Modelo 1 ─▶ Errores ─▶ Modelo 2 ─▶ Errores ─▶ ...  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Random Forest

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: RANDOM FOREST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  "Bosque" de árboles de decisión que votan.                     │
│                                                                 │
│  DOS NIVELES DE ALEATORIEDAD:                                   │
│  1. Bootstrap: Cada árbol entrena con muestra diferente         │
│  2. Random features: Cada división considera subset de features │
│                                                                 │
│  POR QUÉ FUNCIONA:                                              │
│  • Árboles individuales hacen overfitting (alta varianza)       │
│  • Pero sus errores son diferentes (no correlacionados)         │
│  • Al promediar, los errores se cancelan                        │
│                                                                 │
│  HIPERPARÁMETROS CLAVE:                                         │
│  • n_estimators: Número de árboles (más = mejor, hasta un punto)│
│  • max_depth: Profundidad de árboles                            │
│  • max_features: Features por división ('sqrt' para clasif)     │
│  • min_samples_leaf: Tamaño mínimo de hojas                     │
│                                                                 │
│  VENTAJAS:                                                      │
│  ✅ Muy buen rendimiento out-of-the-box                         │
│  ✅ Difícil de hacer overfitting                                │
│  ✅ Importancia de features gratis                              │
│  ✅ Funciona con pocos hiperparámetros                          │
│                                                                 │
│  LIMITACIONES:                                                  │
│  ❌ Menos interpretable que un árbol                            │
│  ❌ Lento para predecir (muchos árboles)                        │
│  ❌ No extrapola bien                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor
from sklearn.datasets import make_classification

# Crear datos
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Entrenar Random Forest
rf = RandomForestClassifier(
    n_estimators=100,      # 100 árboles
    max_depth=10,          # Limitar profundidad
    max_features='sqrt',   # √n features por división
    min_samples_leaf=5,
    n_jobs=-1,             # Usar todos los CPUs
    random_state=42
)
rf.fit(X_train, y_train)

print(f"Accuracy: {rf.score(X_test, y_test):.3f}")

# Importancia de features
importances = rf.feature_importances_
indices = np.argsort(importances)[::-1]

print("\nTop 5 features más importantes:")
for i in range(5):
    print(f"  Feature {indices[i]}: {importances[indices[i]]:.3f}")

# Visualizar importancias
plt.figure(figsize=(10, 6))
plt.bar(range(10), importances[indices[:10]])
plt.xticks(range(10), [f"F{indices[i]}" for i in range(10)])
plt.title("Importancia de Features")
plt.show()
```

### Gradient Boosting y XGBoost

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: GRADIENT BOOSTING                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Construye árboles SECUENCIALMENTE.                             │
│  Cada árbol corrige los errores (residuos) del anterior.        │
│                                                                 │
│  PROCESO:                                                       │
│  1. Entrenar árbol 1 → predice y₁                               │
│  2. Calcular errores: r₁ = y_real - y₁                          │
│  3. Entrenar árbol 2 para predecir r₁                           │
│  4. Predicción total: y₁ + árbol2(x)                            │
│  5. Repetir...                                                  │
│                                                                 │
│  XGBOOST (eXtreme Gradient Boosting):                           │
│  Implementación optimizada con:                                 │
│  • Regularización L1 y L2 incorporada                           │
│  • Manejo de valores faltantes                                  │
│  • Paralelización eficiente                                     │
│  • Poda de árboles inteligente                                  │
│                                                                 │
│  CUÁNDO USAR:                                                   │
│  ✅ Datos tabulares (el rey de Kaggle)                          │
│  ✅ Cuando necesitas máximo rendimiento                         │
│  ✅ Features mixtas (numéricas y categóricas)                   │
│                                                                 │
│  CUIDADO:                                                       │
│  ⚠️ Más propenso a overfitting que Random Forest                │
│  ⚠️ Más sensible a hiperparámetros                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.ensemble import GradientBoostingClassifier
# pip install xgboost
import xgboost as xgb

# Sklearn Gradient Boosting
gb = GradientBoostingClassifier(
    n_estimators=100,
    learning_rate=0.1,     # Cuánto contribuye cada árbol
    max_depth=3,           # Árboles poco profundos
    random_state=42
)
gb.fit(X_train, y_train)
print(f"Gradient Boosting Accuracy: {gb.score(X_test, y_test):.3f}")

# XGBoost (más rápido y potente)
xgb_model = xgb.XGBClassifier(
    n_estimators=100,
    learning_rate=0.1,
    max_depth=3,
    use_label_encoder=False,
    eval_metric='logloss',
    random_state=42
)
xgb_model.fit(X_train, y_train)
print(f"XGBoost Accuracy: {xgb_model.score(X_test, y_test):.3f}")

# Early stopping (detener cuando no mejora)
xgb_model_es = xgb.XGBClassifier(
    n_estimators=1000,  # Muchos, pero early stopping
    learning_rate=0.1,
    max_depth=3,
    early_stopping_rounds=10,
    random_state=42
)
xgb_model_es.fit(
    X_train, y_train,
    eval_set=[(X_test, y_test)],
    verbose=False
)
print(f"XGBoost (early stop) - Mejores iteraciones: {xgb_model_es.best_iteration}")
```

---

## Support Vector Machines (SVM)

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: SUPPORT VECTOR MACHINES                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Encuentra el HIPERPLANO que mejor separa las clases            │
│  maximizando el MARGEN (distancia a los puntos más cercanos).   │
│                                                                 │
│  VISUALIZACIÓN 2D:                                              │
│                                                                 │
│       Clase A (●)              Clase B (○)                      │
│                                                                 │
│       ●  ●                        ○  ○                          │
│          ●      margen            ○                             │
│       ●     ←──│───────│──→   ○   ○                             │
│         ●      │  SV   │      ○                                 │
│       ●        │───────│        ○  ○                            │
│                                                                 │
│  SV = Support Vectors (puntos en el margen, los que importan)   │
│                                                                 │
│  KERNEL TRICK (para datos no separables linealmente):           │
│  Proyecta datos a dimensión superior donde SÍ son separables    │
│                                                                 │
│     2D:  No separable     3D: Separable con plano               │
│     ○ ● ● ○              ○         ○                            │
│     ● ○ ○ ●             ─●─────────●─                           │
│                         ○           ○                           │
│                                                                 │
│  KERNELS COMUNES:                                               │
│  • 'linear': Separación lineal                                  │
│  • 'rbf' (Radial Basis): El más usado, captura no linealidad    │
│  • 'poly': Polinomial                                           │
│                                                                 │
│  HIPERPARÁMETROS:                                               │
│  • C: Penalización por errores (C alto = menos errores, overfít)│
│  • gamma: Radio de influencia de puntos (gamma alto = overfít)  │
│                                                                 │
│  VENTAJAS:                                                      │
│  ✅ Efectivo en alta dimensionalidad                            │
│  ✅ Funciona bien con margen claro                              │
│  ✅ Robusto a outliers (solo usa support vectors)               │
│                                                                 │
│  LIMITACIONES:                                                  │
│  ❌ Lento con muchos datos (O(n²) o peor)                       │
│  ❌ Requiere escalado                                           │
│  ❌ No da probabilidades directamente                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.svm import SVC, SVR
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

# SVM requiere escalado
svm_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(kernel='rbf', C=1.0, gamma='scale', random_state=42))
])

svm_pipeline.fit(X_train, y_train)
print(f"SVM Accuracy: {svm_pipeline.score(X_test, y_test):.3f}")

# Comparar kernels
kernels = ['linear', 'rbf', 'poly']
for kernel in kernels:
    svm = Pipeline([
        ('scaler', StandardScaler()),
        ('svm', SVC(kernel=kernel, random_state=42))
    ])
    svm.fit(X_train, y_train)
    print(f"  {kernel}: {svm.score(X_test, y_test):.3f}")

# SVM con probabilidades
svm_prob = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(kernel='rbf', probability=True, random_state=42))
])
svm_prob.fit(X_train, y_train)
probs = svm_prob.predict_proba(X_test[:5])
print(f"\nProbabilidades:\n{probs}")
```

---

## K-Nearest Neighbors (KNN)

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: K-NEAREST NEIGHBORS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  "Dime con quién andas y te diré quién eres"                    │
│  Clasifica según los K vecinos más cercanos.                    │
│                                                                 │
│  PROCESO:                                                       │
│  1. Recibe punto nuevo (?)                                      │
│  2. Calcula distancia a todos los puntos de entrenamiento       │
│  3. Encuentra los K más cercanos                                │
│  4. Vota: la clase mayoritaria gana                             │
│                                                                 │
│  EJEMPLO con K=3:                                               │
│                                                                 │
│       ●       ○                                                 │
│         ● ○ [?]    ← Vecinos: 2 ●, 1 ○ → Predice ●             │
│       ●   ○   ○                                                 │
│                                                                 │
│  DISTANCIAS:                                                    │
│  • Euclidiana: √(Σ(xᵢ - yᵢ)²)  (la más común)                   │
│  • Manhattan: Σ|xᵢ - yᵢ|                                        │
│  • Minkowski: generalización                                    │
│                                                                 │
│  K (número de vecinos):                                         │
│  • K pequeño (1-3): Sensible a ruido, puede overfittear         │
│  • K grande: Más suave, puede underfittear                      │
│  • Regla: K impar para evitar empates                           │
│                                                                 │
│  VENTAJAS:                                                      │
│  ✅ Simple e intuitivo                                          │
│  ✅ No hay entrenamiento (lazy learner)                         │
│  ✅ Se adapta naturalmente a nuevos datos                       │
│                                                                 │
│  LIMITACIONES:                                                  │
│  ❌ Lento en predicción (calcula todas las distancias)          │
│  ❌ Sufre "maldición de dimensionalidad"                        │
│  ❌ Sensible a features irrelevantes                            │
│  ❌ REQUIERE ESCALADO (distancias)                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

# KNN REQUIERE escalado
knn_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('knn', KNeighborsClassifier(n_neighbors=5))
])

knn_pipeline.fit(X_train, y_train)
print(f"KNN Accuracy: {knn_pipeline.score(X_test, y_test):.3f}")

# Encontrar mejor K
from sklearn.model_selection import cross_val_score

best_k = 1
best_score = 0

for k in range(1, 21, 2):  # Solo impares
    knn = Pipeline([
        ('scaler', StandardScaler()),
        ('knn', KNeighborsClassifier(n_neighbors=k))
    ])
    scores = cross_val_score(knn, X_train, y_train, cv=5)
    mean_score = scores.mean()
    
    if mean_score > best_score:
        best_score = mean_score
        best_k = k

print(f"\nMejor K: {best_k} (CV Score: {best_score:.3f})")
```

---

## Comparación de Algoritmos

```
┌─────────────────────────────────────────────────────────────────┐
│              GUÍA RÁPIDA DE SELECCIÓN                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ESCENARIO                         │  ALGORITMO SUGERIDO        │
│  ─────────────────────────────────────────────────────────────  │
│  Baseline rápido                   │  Logistic/Linear Reg       │
│  Interpretabilidad máxima          │  Decision Tree, Logistic   │
│  Máximo rendimiento (tabular)      │  XGBoost, Random Forest    │
│  Datos pequeños                    │  SVM, KNN                  │
│  Datos muy grandes                 │  Random Forest, XGBoost    │
│  Clasificación de texto            │  Naive Bayes, Logistic     │
│  Muchas features irrelevantes      │  Lasso, Random Forest      │
│  Relaciones no lineales            │  Random Forest, XGBoost    │
│                                                                 │
│  FLUJO DE TRABAJO TÍPICO:                                       │
│                                                                 │
│  1. Empezar con baseline simple (Logistic/Linear)               │
│  2. Probar Random Forest (robusto, pocos hiperparámetros)       │
│  3. Si necesitas más: XGBoost con tuning                        │
│  4. Si necesitas interpretabilidad: Tree o Logistic             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# COMPARACIÓN AUTOMÁTICA DE MODELOS
# ========================================
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.naive_bayes import GaussianNB
from sklearn.model_selection import cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline

# Modelos a comparar
models = {
    'Logistic Regression': LogisticRegression(random_state=42),
    'Decision Tree': DecisionTreeClassifier(max_depth=5, random_state=42),
    'Random Forest': RandomForestClassifier(n_estimators=100, random_state=42),
    'Gradient Boosting': GradientBoostingClassifier(random_state=42),
    'SVM': SVC(random_state=42),
    'KNN': KNeighborsClassifier(),
    'Naive Bayes': GaussianNB()
}

print("Comparación de Modelos (5-fold CV):\n")
print(f"{'Modelo':<25} {'Accuracy':<12} {'Std':<8}")
print("-" * 45)

results = []
for name, model in models.items():
    # SVM y KNN necesitan escalado
    if name in ['SVM', 'KNN']:
        model = Pipeline([
            ('scaler', StandardScaler()),
            ('model', model)
        ])
    
    scores = cross_val_score(model, X, y, cv=5, scoring='accuracy')
    results.append((name, scores.mean(), scores.std()))
    print(f"{name:<25} {scores.mean():.3f}        ±{scores.std():.3f}")

# Mejor modelo
best = max(results, key=lambda x: x[1])
print(f"\n🏆 Mejor modelo: {best[0]} ({best[1]:.3f})")
```

---

## 🏷️ Tags

#programming #machinelearning #python #algorithms #datascience

---

## 📚 Ver También

- [[ML Fundamentals Guide|Fundamentos de ML]]
- [[Deep Learning Guide|Deep Learning - Guía Completa]]
- [[ML Projects Guide|Proyectos Prácticos]]
