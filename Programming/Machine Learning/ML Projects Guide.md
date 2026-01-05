# Machine Learning - Proyectos Prácticos

> **Proyectos completos paso a paso para practicar ML**
> Última actualización: Enero 2026

---

## Tabla de Contenidos

1. [[#Proyecto 1 - Predicción de Precios de Casas|Regresión: Precios de Casas]]
2. [[#Proyecto 2 - Clasificación de Sentimientos|NLP: Análisis de Sentimientos]]
3. [[#Proyecto 3 - Clasificación de Imágenes|CNN: Clasificador de Imágenes]]
4. [[#Proyecto 4 - Sistema de Recomendación|RecSys: Recomendaciones]]
5. [[#Herramientas y Setup|Setup del Entorno]]

---

## Herramientas y Setup

### Instalación del Entorno

```bash
# Crear entorno virtual
python -m venv ml_env
source ml_env/bin/activate  # Linux/Mac
ml_env\Scripts\activate     # Windows

# Librerías fundamentales
pip install numpy pandas matplotlib seaborn scikit-learn

# Deep Learning
pip install torch torchvision torchaudio
pip install transformers datasets

# Extras útiles
pip install jupyter notebook
pip install xgboost lightgbm
pip install plotly
```

### Librerías Principales

```python
# ========================================
# IMPORTS ESTÁNDAR PARA ML
# ========================================

# Manipulación de datos
import numpy as np
import pandas as pd

# Visualización
import matplotlib.pyplot as plt
import seaborn as sns
plt.style.use('seaborn-v0_8-whitegrid')

# Sklearn - Preprocesamiento
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler, LabelEncoder, OneHotEncoder
from sklearn.impute import SimpleImputer

# Sklearn - Modelos
from sklearn.linear_model import LogisticRegression, LinearRegression
from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor
from sklearn.ensemble import GradientBoostingClassifier

# Sklearn - Métricas
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    mean_squared_error, r2_score, classification_report,
    confusion_matrix, roc_auc_score, roc_curve
)

# Sklearn - Pipelines
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer

# Configuración
import warnings
warnings.filterwarnings('ignore')

# Reproducibilidad
RANDOM_STATE = 42
np.random.seed(RANDOM_STATE)
```

---

## Proyecto 1 - Predicción de Precios de Casas

### Objetivo
Predecir el precio de venta de casas basándose en sus características.

```python
# ========================================
# PROYECTO: PREDICCIÓN DE PRECIOS
# ========================================
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LinearRegression, Ridge
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error

# ----------------------------------------
# 1. CARGAR Y EXPLORAR DATOS
# ----------------------------------------

# Crear dataset sintético (en la práctica usarías datos reales)
np.random.seed(42)
n_samples = 1000

data = pd.DataFrame({
    'area_m2': np.random.uniform(50, 300, n_samples),
    'habitaciones': np.random.randint(1, 6, n_samples),
    'baños': np.random.randint(1, 4, n_samples),
    'antiguedad': np.random.randint(0, 50, n_samples),
    'tiene_garage': np.random.choice([0, 1], n_samples),
    'tiene_jardin': np.random.choice([0, 1], n_samples),
    'piso': np.random.randint(0, 20, n_samples),
    'distancia_centro_km': np.random.uniform(0.5, 30, n_samples),
    'zona': np.random.choice(['norte', 'sur', 'este', 'oeste'], n_samples)
})

# Crear precio (relación realista)
data['precio'] = (
    data['area_m2'] * 2000 +
    data['habitaciones'] * 15000 +
    data['baños'] * 10000 -
    data['antiguedad'] * 500 +
    data['tiene_garage'] * 20000 +
    data['tiene_jardin'] * 15000 -
    data['distancia_centro_km'] * 1500 +
    np.random.normal(0, 20000, n_samples)
)

print("="*50)
print("EXPLORACIÓN DE DATOS")
print("="*50)
print(f"\nShape: {data.shape}")
print(f"\nPrimeras filas:\n{data.head()}")
print(f"\nEstadísticas:\n{data.describe()}")
print(f"\nValores nulos:\n{data.isnull().sum()}")

# ----------------------------------------
# 2. ANÁLISIS EXPLORATORIO (EDA)
# ----------------------------------------

fig, axes = plt.subplots(2, 3, figsize=(15, 10))

# Distribución del precio
axes[0, 0].hist(data['precio'], bins=30, edgecolor='black')
axes[0, 0].set_title('Distribución de Precios')
axes[0, 0].set_xlabel('Precio')

# Precio vs Area
axes[0, 1].scatter(data['area_m2'], data['precio'], alpha=0.5)
axes[0, 1].set_title('Precio vs Área')
axes[0, 1].set_xlabel('Área (m²)')
axes[0, 1].set_ylabel('Precio')

# Precio vs Habitaciones
data.boxplot(column='precio', by='habitaciones', ax=axes[0, 2])
axes[0, 2].set_title('Precio por Habitaciones')

# Matriz de correlación
numeric_cols = data.select_dtypes(include=[np.number]).columns
corr_matrix = data[numeric_cols].corr()
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', ax=axes[1, 0], fmt='.2f')
axes[1, 0].set_title('Correlación')

# Precio por zona
data.groupby('zona')['precio'].mean().plot(kind='bar', ax=axes[1, 1])
axes[1, 1].set_title('Precio Promedio por Zona')
axes[1, 1].tick_params(axis='x', rotation=45)

# Precio vs Antigüedad
axes[1, 2].scatter(data['antiguedad'], data['precio'], alpha=0.5)
axes[1, 2].set_title('Precio vs Antigüedad')
axes[1, 2].set_xlabel('Antigüedad (años)')

plt.tight_layout()
plt.show()

# ----------------------------------------
# 3. PREPARACIÓN DE DATOS
# ----------------------------------------

# Separar features y target
X = data.drop('precio', axis=1)
y = data['precio']

# One-hot encoding para zona
X = pd.get_dummies(X, columns=['zona'], prefix='zona')

# División train/test
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Escalar features numéricas
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

print("\n" + "="*50)
print("DATOS PREPARADOS")
print("="*50)
print(f"Train: {X_train.shape}, Test: {X_test.shape}")
print(f"Features: {list(X.columns)}")

# ----------------------------------------
# 4. ENTRENAR Y COMPARAR MODELOS
# ----------------------------------------

models = {
    'Linear Regression': LinearRegression(),
    'Ridge': Ridge(alpha=1.0),
    'Random Forest': RandomForestRegressor(n_estimators=100, random_state=42),
    'Gradient Boosting': GradientBoostingRegressor(n_estimators=100, random_state=42)
}

print("\n" + "="*50)
print("COMPARACIÓN DE MODELOS")
print("="*50)

results = []
for name, model in models.items():
    # Cross-validation
    cv_scores = cross_val_score(model, X_train_scaled, y_train, cv=5, scoring='r2')
    
    # Entrenar y predecir
    model.fit(X_train_scaled, y_train)
    y_pred = model.predict(X_test_scaled)
    
    # Métricas
    r2 = r2_score(y_test, y_pred)
    rmse = np.sqrt(mean_squared_error(y_test, y_pred))
    mae = mean_absolute_error(y_test, y_pred)
    
    results.append({
        'Modelo': name,
        'R² CV': f"{cv_scores.mean():.3f} ± {cv_scores.std():.3f}",
        'R² Test': f"{r2:.3f}",
        'RMSE': f"{rmse:,.0f}",
        'MAE': f"{mae:,.0f}"
    })
    
results_df = pd.DataFrame(results)
print(results_df.to_string(index=False))

# ----------------------------------------
# 5. ANALIZAR MEJOR MODELO
# ----------------------------------------

best_model = models['Random Forest']
best_model.fit(X_train_scaled, y_train)
y_pred = best_model.predict(X_test_scaled)

# Importancia de features
feature_importance = pd.DataFrame({
    'feature': X.columns,
    'importance': best_model.feature_importances_
}).sort_values('importance', ascending=False)

print("\n" + "="*50)
print("IMPORTANCIA DE FEATURES (Random Forest)")
print("="*50)
print(feature_importance.to_string(index=False))

# Visualizar predicciones vs reales
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Predicciones vs Reales
axes[0].scatter(y_test, y_pred, alpha=0.5)
axes[0].plot([y_test.min(), y_test.max()], [y_test.min(), y_test.max()], 'r--', lw=2)
axes[0].set_xlabel('Precio Real')
axes[0].set_ylabel('Precio Predicho')
axes[0].set_title('Predicciones vs Valores Reales')

# Residuos
residuals = y_test - y_pred
axes[1].hist(residuals, bins=30, edgecolor='black')
axes[1].axvline(x=0, color='r', linestyle='--')
axes[1].set_xlabel('Residuo')
axes[1].set_title('Distribución de Residuos')

plt.tight_layout()
plt.show()

# ----------------------------------------
# 6. PREDICCIÓN CON NUEVA CASA
# ----------------------------------------

print("\n" + "="*50)
print("PREDICCIÓN DE NUEVA CASA")
print("="*50)

nueva_casa = pd.DataFrame({
    'area_m2': [150],
    'habitaciones': [3],
    'baños': [2],
    'antiguedad': [5],
    'tiene_garage': [1],
    'tiene_jardin': [1],
    'piso': [3],
    'distancia_centro_km': [8],
    'zona_este': [0],
    'zona_norte': [1],
    'zona_oeste': [0],
    'zona_sur': [0]
})

nueva_casa_scaled = scaler.transform(nueva_casa)
precio_predicho = best_model.predict(nueva_casa_scaled)[0]

print(f"\nCaracterísticas:")
print(f"  - Área: 150 m²")
print(f"  - Habitaciones: 3")
print(f"  - Baños: 2")
print(f"  - Antigüedad: 5 años")
print(f"  - Con garage y jardín")
print(f"  - Zona norte, 8km del centro")
print(f"\n💰 Precio Predicho: ${precio_predicho:,.0f}")
```

---

## Proyecto 2 - Clasificación de Sentimientos

### Objetivo
Clasificar reseñas de películas como positivas o negativas.

```python
# ========================================
# PROYECTO: ANÁLISIS DE SENTIMIENTOS
# ========================================
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.naive_bayes import MultinomialNB
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix
import re

# ----------------------------------------
# 1. CREAR/CARGAR DATOS
# ----------------------------------------

# Dataset de ejemplo (en práctica: IMDB, Yelp, Twitter, etc.)
reviews = [
    # Positivas
    "Esta película es increíble, la mejor que he visto",
    "Excelente actuación y una historia conmovedora",
    "Me encantó cada minuto, muy recomendada",
    "Una obra maestra del cine contemporáneo",
    "Los efectos especiales son impresionantes",
    "La mejor película del año sin duda",
    "Historia emotiva con un final perfecto",
    "Gran dirección y fotografía espectacular",
    "Actuaciones brillantes de todo el elenco",
    "Me hizo reír y llorar, película completa",
    "Increíble desarrollo de personajes",
    "Visualmente impresionante y bien escrita",
    
    # Negativas
    "Película aburrida y predecible",
    "No vale la pena perder el tiempo",
    "Pésimas actuaciones y guión terrible",
    "La peor película que he visto en años",
    "Historia sin sentido y mal ejecutada",
    "Totalmente decepcionante y aburrida",
    "No la recomiendo para nada",
    "Desperdicio de tiempo y dinero",
    "Efectos baratos y actuación horrible",
    "Final absurdo que arruina todo",
    "Película vacía y sin emoción",
    "Muy mala, evítenla a toda costa"
]

labels = [1]*12 + [0]*12  # 1=positivo, 0=negativo

# Convertir a DataFrame
data = pd.DataFrame({'text': reviews, 'sentiment': labels})
print("Dataset:")
print(data.head(10))
print(f"\nDistribución: {data['sentiment'].value_counts().to_dict()}")

# ----------------------------------------
# 2. PREPROCESAMIENTO DE TEXTO
# ----------------------------------------

def preprocess_text(text):
    """Limpieza básica de texto"""
    # Minúsculas
    text = text.lower()
    # Eliminar caracteres especiales
    text = re.sub(r'[^\w\s]', '', text)
    # Eliminar números
    text = re.sub(r'\d+', '', text)
    # Eliminar espacios extra
    text = ' '.join(text.split())
    return text

data['text_clean'] = data['text'].apply(preprocess_text)
print("\nTexto limpio:")
print(data[['text', 'text_clean']].head())

# ----------------------------------------
# 3. VECTORIZACIÓN (TF-IDF)
# ----------------------------------------

# División
X_train, X_test, y_train, y_test = train_test_split(
    data['text_clean'], data['sentiment'], 
    test_size=0.25, random_state=42
)

# TF-IDF Vectorizer
vectorizer = TfidfVectorizer(
    max_features=1000,      # Máximo vocabulario
    ngram_range=(1, 2),     # Unigramas y bigramas
    min_df=1,               # Mínimo documentos
    stop_words=None         # Podemos añadir stopwords en español
)

X_train_tfidf = vectorizer.fit_transform(X_train)
X_test_tfidf = vectorizer.transform(X_test)

print(f"\nVocabulario: {len(vectorizer.vocabulary_)} términos")
print(f"Shape train: {X_train_tfidf.shape}")

# Top términos por TF-IDF
feature_names = vectorizer.get_feature_names_out()
print(f"\nAlgunos términos: {list(feature_names[:20])}")

# ----------------------------------------
# 4. ENTRENAR MODELOS
# ----------------------------------------

models = {
    'Logistic Regression': LogisticRegression(random_state=42),
    'Naive Bayes': MultinomialNB(),
    'Random Forest': RandomForestClassifier(n_estimators=100, random_state=42)
}

print("\n" + "="*50)
print("RESULTADOS")
print("="*50)

for name, model in models.items():
    model.fit(X_train_tfidf, y_train)
    y_pred = model.predict(X_test_tfidf)
    
    print(f"\n{name}:")
    print(classification_report(y_test, y_pred, 
                                target_names=['Negativo', 'Positivo']))

# ----------------------------------------
# 5. ANALIZAR MODELO
# ----------------------------------------

# Usar Logistic Regression para interpretabilidad
best_model = models['Logistic Regression']

# Palabras más influyentes
coef = best_model.coef_[0]
top_positive = np.argsort(coef)[-10:]
top_negative = np.argsort(coef)[:10]

print("\n" + "="*50)
print("PALABRAS MÁS INFLUYENTES")
print("="*50)
print("\n🟢 Palabras positivas:")
for idx in top_positive[::-1]:
    print(f"  {feature_names[idx]}: {coef[idx]:.3f}")

print("\n🔴 Palabras negativas:")
for idx in top_negative:
    print(f"  {feature_names[idx]}: {coef[idx]:.3f}")

# ----------------------------------------
# 6. PREDECIR NUEVAS RESEÑAS
# ----------------------------------------

print("\n" + "="*50)
print("PREDICCIONES NUEVAS")
print("="*50)

new_reviews = [
    "Una película fantástica que disfruté mucho",
    "No me gustó nada, muy aburrida",
    "Regular, tiene cosas buenas y malas"
]

for review in new_reviews:
    review_clean = preprocess_text(review)
    review_tfidf = vectorizer.transform([review_clean])
    
    prediction = best_model.predict(review_tfidf)[0]
    probability = best_model.predict_proba(review_tfidf)[0]
    
    sentiment = "😊 Positivo" if prediction == 1 else "😞 Negativo"
    confidence = max(probability) * 100
    
    print(f"\n\"{review}\"")
    print(f"  → {sentiment} (Confianza: {confidence:.1f}%)")
```

---

## Proyecto 3 - Clasificación de Imágenes

### Objetivo
Clasificar imágenes usando CNN con PyTorch.

```python
# ========================================
# PROYECTO: CLASIFICACIÓN DE IMÁGENES (CNN)
# ========================================
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
import torchvision
import torchvision.transforms as transforms
import matplotlib.pyplot as plt
import numpy as np

# Configuración
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Usando: {device}")

# ----------------------------------------
# 1. CARGAR DATOS (CIFAR-10)
# ----------------------------------------

# Data augmentation para entrenamiento
train_transform = transforms.Compose([
    transforms.RandomHorizontalFlip(),
    transforms.RandomRotation(10),
    transforms.RandomCrop(32, padding=4),
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), 
                         (0.2023, 0.1994, 0.2010))
])

test_transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), 
                         (0.2023, 0.1994, 0.2010))
])

# Descargar CIFAR-10
train_dataset = torchvision.datasets.CIFAR10(
    root='./data', train=True, download=True, transform=train_transform
)
test_dataset = torchvision.datasets.CIFAR10(
    root='./data', train=False, download=True, transform=test_transform
)

# DataLoaders
train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True, num_workers=2)
test_loader = DataLoader(test_dataset, batch_size=64, shuffle=False, num_workers=2)

classes = ('avión', 'auto', 'pájaro', 'gato', 'venado', 
           'perro', 'rana', 'caballo', 'barco', 'camión')

print(f"Train: {len(train_dataset)}, Test: {len(test_dataset)}")
print(f"Clases: {classes}")

# Visualizar algunas imágenes
def show_images(loader, classes, n=8):
    images, labels = next(iter(loader))
    fig, axes = plt.subplots(1, n, figsize=(15, 2))
    for i in range(n):
        img = images[i].permute(1, 2, 0).numpy()
        img = img * np.array([0.2023, 0.1994, 0.2010]) + np.array([0.4914, 0.4822, 0.4465])
        img = np.clip(img, 0, 1)
        axes[i].imshow(img)
        axes[i].set_title(classes[labels[i]])
        axes[i].axis('off')
    plt.show()

show_images(train_loader, classes)

# ----------------------------------------
# 2. DEFINIR CNN
# ----------------------------------------

class CNN(nn.Module):
    def __init__(self, num_classes=10):
        super(CNN, self).__init__()
        
        self.features = nn.Sequential(
            # Bloque 1
            nn.Conv2d(3, 32, kernel_size=3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.Conv2d(32, 32, kernel_size=3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),
            nn.Dropout2d(0.25),
            
            # Bloque 2
            nn.Conv2d(32, 64, kernel_size=3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.Conv2d(64, 64, kernel_size=3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),
            nn.Dropout2d(0.25),
            
            # Bloque 3
            nn.Conv2d(64, 128, kernel_size=3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(),
            nn.Conv2d(128, 128, kernel_size=3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),
            nn.Dropout2d(0.25),
        )
        
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(128 * 4 * 4, 512),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(512, num_classes)
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x

model = CNN(num_classes=10).to(device)
print(model)

# Contar parámetros
total_params = sum(p.numel() for p in model.parameters())
trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"\nParámetros totales: {total_params:,}")
print(f"Parámetros entrenables: {trainable_params:,}")

# ----------------------------------------
# 3. ENTRENAR
# ----------------------------------------

criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001, weight_decay=1e-4)
scheduler = optim.lr_scheduler.ReduceLROnPlateau(optimizer, patience=3, factor=0.5)

def train_epoch(model, loader, criterion, optimizer, device):
    model.train()
    running_loss = 0.0
    correct = 0
    total = 0
    
    for images, labels in loader:
        images, labels = images.to(device), labels.to(device)
        
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
        
        running_loss += loss.item()
        _, predicted = outputs.max(1)
        total += labels.size(0)
        correct += predicted.eq(labels).sum().item()
    
    return running_loss / len(loader), 100. * correct / total

def evaluate(model, loader, criterion, device):
    model.eval()
    running_loss = 0.0
    correct = 0
    total = 0
    
    with torch.no_grad():
        for images, labels in loader:
            images, labels = images.to(device), labels.to(device)
            outputs = model(images)
            loss = criterion(outputs, labels)
            
            running_loss += loss.item()
            _, predicted = outputs.max(1)
            total += labels.size(0)
            correct += predicted.eq(labels).sum().item()
    
    return running_loss / len(loader), 100. * correct / total

# Entrenamiento
epochs = 20
history = {'train_loss': [], 'train_acc': [], 'val_loss': [], 'val_acc': []}

print("\nEntrenando...")
for epoch in range(epochs):
    train_loss, train_acc = train_epoch(model, train_loader, criterion, optimizer, device)
    val_loss, val_acc = evaluate(model, test_loader, criterion, device)
    
    scheduler.step(val_loss)
    
    history['train_loss'].append(train_loss)
    history['train_acc'].append(train_acc)
    history['val_loss'].append(val_loss)
    history['val_acc'].append(val_acc)
    
    print(f"Epoch {epoch+1}/{epochs}")
    print(f"  Train Loss: {train_loss:.4f}, Acc: {train_acc:.2f}%")
    print(f"  Val Loss: {val_loss:.4f}, Acc: {val_acc:.2f}%")

# ----------------------------------------
# 4. VISUALIZAR RESULTADOS
# ----------------------------------------

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Loss
axes[0].plot(history['train_loss'], label='Train')
axes[0].plot(history['val_loss'], label='Validation')
axes[0].set_xlabel('Epoch')
axes[0].set_ylabel('Loss')
axes[0].set_title('Loss durante entrenamiento')
axes[0].legend()

# Accuracy
axes[1].plot(history['train_acc'], label='Train')
axes[1].plot(history['val_acc'], label='Validation')
axes[1].set_xlabel('Epoch')
axes[1].set_ylabel('Accuracy (%)')
axes[1].set_title('Accuracy durante entrenamiento')
axes[1].legend()

plt.tight_layout()
plt.show()

# ----------------------------------------
# 5. MATRIZ DE CONFUSIÓN
# ----------------------------------------

from sklearn.metrics import confusion_matrix
import seaborn as sns

model.eval()
all_preds = []
all_labels = []

with torch.no_grad():
    for images, labels in test_loader:
        images = images.to(device)
        outputs = model(images)
        _, preds = outputs.max(1)
        all_preds.extend(preds.cpu().numpy())
        all_labels.extend(labels.numpy())

cm = confusion_matrix(all_labels, all_preds)
plt.figure(figsize=(10, 8))
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues',
            xticklabels=classes, yticklabels=classes)
plt.xlabel('Predicho')
plt.ylabel('Real')
plt.title('Matriz de Confusión')
plt.show()

# Accuracy por clase
print("\nAccuracy por clase:")
for i, cls in enumerate(classes):
    class_correct = cm[i, i]
    class_total = cm[i].sum()
    print(f"  {cls}: {100 * class_correct / class_total:.1f}%")
```

---

## Proyecto 4 - Sistema de Recomendación

### Objetivo
Crear un sistema de recomendación de películas.

```python
# ========================================
# PROYECTO: SISTEMA DE RECOMENDACIÓN
# ========================================
import numpy as np
import pandas as pd
from sklearn.metrics.pairwise import cosine_similarity
from sklearn.feature_extraction.text import TfidfVectorizer

# ----------------------------------------
# 1. CREAR DATOS DE EJEMPLO
# ----------------------------------------

# Películas con metadata
movies = pd.DataFrame({
    'movie_id': range(1, 11),
    'title': [
        'The Matrix', 'Inception', 'Interstellar', 
        'The Dark Knight', 'Pulp Fiction',
        'Forrest Gump', 'The Shawshank Redemption', 
        'Fight Club', 'The Godfather', 'Goodfellas'
    ],
    'genre': [
        'sci-fi action', 'sci-fi thriller', 'sci-fi drama',
        'action crime', 'crime drama',
        'drama romance', 'drama', 
        'drama thriller', 'crime drama', 'crime drama'
    ],
    'director': [
        'Wachowski', 'Nolan', 'Nolan',
        'Nolan', 'Tarantino',
        'Zemeckis', 'Darabont',
        'Fincher', 'Coppola', 'Scorsese'
    ],
    'year': [1999, 2010, 2014, 2008, 1994, 1994, 1994, 1999, 1972, 1990]
})

# Ratings de usuarios
np.random.seed(42)
n_users = 20
n_movies = 10

# Crear matriz de ratings (algunos NaN)
ratings_matrix = np.random.choice([np.nan, 1, 2, 3, 4, 5], 
                                   size=(n_users, n_movies),
                                   p=[0.5, 0.1, 0.1, 0.1, 0.1, 0.1])
ratings_df = pd.DataFrame(ratings_matrix, 
                          columns=movies['title'],
                          index=[f'User_{i}' for i in range(n_users)])

print("Películas:")
print(movies)
print("\nMatriz de Ratings (muestra):")
print(ratings_df.head())

# ----------------------------------------
# 2. RECOMENDACIÓN BASADA EN CONTENIDO
# ----------------------------------------

print("\n" + "="*50)
print("RECOMENDACIÓN BASADA EN CONTENIDO")
print("="*50)

# Combinar features de contenido
movies['content'] = movies['genre'] + ' ' + movies['director']

# Vectorizar contenido
tfidf = TfidfVectorizer(stop_words='english')
content_matrix = tfidf.fit_transform(movies['content'])

# Similitud entre películas
content_similarity = cosine_similarity(content_matrix)
content_sim_df = pd.DataFrame(content_similarity,
                              index=movies['title'],
                              columns=movies['title'])

def get_content_recommendations(movie_title, n=5):
    """Recomienda películas similares basado en contenido"""
    if movie_title not in content_sim_df.columns:
        return f"Película '{movie_title}' no encontrada"
    
    # Obtener similitudes
    similarities = content_sim_df[movie_title]
    
    # Ordenar y excluir la película misma
    similar_movies = similarities.sort_values(ascending=False)[1:n+1]
    
    return similar_movies

# Ejemplo
print("\nPelículas similares a 'Inception':")
recs = get_content_recommendations('Inception')
for title, score in recs.items():
    print(f"  {title}: {score:.3f}")

print("\nPelículas similares a 'The Godfather':")
recs = get_content_recommendations('The Godfather')
for title, score in recs.items():
    print(f"  {title}: {score:.3f}")

# ----------------------------------------
# 3. FILTRADO COLABORATIVO
# ----------------------------------------

print("\n" + "="*50)
print("FILTRADO COLABORATIVO")
print("="*50)

# Rellenar NaN con promedio del usuario para calcular similitud
ratings_filled = ratings_df.apply(lambda x: x.fillna(x.mean()), axis=1)

# Similitud entre usuarios
user_similarity = cosine_similarity(ratings_filled)
user_sim_df = pd.DataFrame(user_similarity,
                           index=ratings_df.index,
                           columns=ratings_df.index)

def get_user_recommendations(user_id, n=5):
    """Recomienda películas basado en usuarios similares"""
    # Películas no vistas por el usuario
    user_ratings = ratings_df.loc[user_id]
    unwatched = user_ratings[user_ratings.isna()].index.tolist()
    
    if not unwatched:
        return "Usuario ha visto todas las películas"
    
    # Encontrar usuarios similares
    similar_users = user_sim_df[user_id].sort_values(ascending=False)[1:6]
    
    # Predecir ratings para películas no vistas
    predictions = {}
    for movie in unwatched:
        movie_ratings = []
        weights = []
        
        for similar_user, similarity in similar_users.items():
            rating = ratings_df.loc[similar_user, movie]
            if not np.isnan(rating):
                movie_ratings.append(rating)
                weights.append(similarity)
        
        if movie_ratings:
            # Weighted average
            predictions[movie] = np.average(movie_ratings, weights=weights)
    
    # Ordenar por predicción
    sorted_predictions = sorted(predictions.items(), key=lambda x: x[1], reverse=True)
    return sorted_predictions[:n]

# Ejemplo
print(f"\nRecomendaciones para User_0:")
recs = get_user_recommendations('User_0')
for movie, predicted_rating in recs:
    print(f"  {movie}: {predicted_rating:.2f} ⭐")

# ----------------------------------------
# 4. SISTEMA HÍBRIDO
# ----------------------------------------

print("\n" + "="*50)
print("SISTEMA HÍBRIDO")
print("="*50)

def hybrid_recommendations(user_id, n=5, content_weight=0.3):
    """Combina filtrado colaborativo y basado en contenido"""
    
    # Obtener recomendaciones colaborativas
    collab_recs = get_user_recommendations(user_id, n=10)
    if isinstance(collab_recs, str):
        return collab_recs
    
    collab_dict = {movie: score for movie, score in collab_recs}
    
    # Para cada película, añadir bonus por similitud de contenido
    # con películas que el usuario ha calificado alto
    user_ratings = ratings_df.loc[user_id].dropna()
    liked_movies = user_ratings[user_ratings >= 4].index.tolist()
    
    final_scores = {}
    for movie, collab_score in collab_dict.items():
        # Score de contenido: similitud promedio con películas gustadas
        content_scores = []
        for liked in liked_movies:
            if movie in content_sim_df.columns and liked in content_sim_df.columns:
                content_scores.append(content_sim_df.loc[movie, liked])
        
        content_score = np.mean(content_scores) if content_scores else 0
        
        # Combinar
        final_scores[movie] = (1 - content_weight) * collab_score + content_weight * (content_score * 5)
    
    # Ordenar
    sorted_recs = sorted(final_scores.items(), key=lambda x: x[1], reverse=True)
    return sorted_recs[:n]

# Ejemplo
print(f"\nRecomendaciones híbridas para User_0:")
recs = hybrid_recommendations('User_0')
for movie, score in recs:
    print(f"  {movie}: {score:.2f}")

# ----------------------------------------
# 5. EVALUACIÓN
# ----------------------------------------

print("\n" + "="*50)
print("EVALUACIÓN DEL SISTEMA")
print("="*50)

def evaluate_recommendations(test_ratings, predictions):
    """Calcula RMSE entre predicciones y ratings reales"""
    errors = []
    for (user, movie), true_rating in test_ratings.items():
        if (user, movie) in predictions:
            pred_rating = predictions[(user, movie)]
            errors.append((true_rating - pred_rating) ** 2)
    
    if errors:
        rmse = np.sqrt(np.mean(errors))
        return rmse
    return None

# Simular evaluación (en práctica: dividir datos en train/test)
print("\nMétricas de evaluación:")
print("  - Coverage: % de items que el sistema puede recomendar")
print("  - Diversity: Variedad en las recomendaciones")
print("  - Novelty: Qué tan 'sorprendentes' son las recomendaciones")
print("  - RMSE: Error en predicción de ratings")
```

---

## Checklist de Proyecto ML

```
┌─────────────────────────────────────────────────────────────────┐
│              CHECKLIST PARA PROYECTOS DE ML                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FASE 1: DEFINICIÓN                                             │
│  □ Definir objetivo de negocio claramente                       │
│  □ Definir métrica de éxito                                     │
│  □ Establecer baseline (modelo simple o regla)                  │
│                                                                 │
│  FASE 2: DATOS                                                  │
│  □ Recolectar/obtener datos                                     │
│  □ EDA (Exploratory Data Analysis)                              │
│  □ Verificar calidad de datos                                   │
│  □ Manejar valores faltantes                                    │
│  □ Manejar outliers                                             │
│  □ Feature engineering                                          │
│  □ Dividir train/val/test                                       │
│                                                                 │
│  FASE 3: MODELADO                                               │
│  □ Empezar con modelo simple (baseline)                         │
│  □ Probar varios algoritmos                                     │
│  □ Cross-validation                                             │
│  □ Hyperparameter tuning                                        │
│  □ Analizar errores                                             │
│                                                                 │
│  FASE 4: EVALUACIÓN                                             │
│  □ Evaluar en test set (UNA vez)                                │
│  □ Comparar con baseline                                        │
│  □ Verificar no hay data leakage                                │
│  □ Interpretar resultados                                       │
│                                                                 │
│  FASE 5: DESPLIEGUE                                             │
│  □ Guardar modelo (pickle, joblib, torch.save)                  │
│  □ Crear pipeline de preprocesamiento                           │
│  □ API o integración                                            │
│  □ Monitoreo en producción                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏷️ Tags

#programming #machinelearning #python #projects #datascience

---

## 📚 Ver También

- [[ML Fundamentals Guide|Fundamentos de ML]]
- [[ML Algorithms Guide|Algoritmos de ML]]
- [[Deep Learning Guide|Deep Learning]]
