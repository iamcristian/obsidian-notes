# Deep Learning - Guía Completa

> **Redes neuronales desde cero hasta arquitecturas avanzadas**
> Última actualización: Enero 2026

---

## Tabla de Contenidos

1. [[#¿Qué es Deep Learning?|Fundamentos]]
2. [[#Perceptrón y Redes Neuronales|Redes Neuronales Básicas]]
3. [[#Backpropagation|Backpropagation]]
4. [[#Redes Convolucionales (CNN)|CNN para Imágenes]]
5. [[#Redes Recurrentes (RNN)|RNN para Secuencias]]
6. [[#Transformers|Transformers (NLP Moderno)]]

---

## ¿Qué es Deep Learning?

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: DEEP LEARNING                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Deep Learning es un subconjunto de ML que usa REDES            │
│  NEURONALES con múltiples capas para aprender representaciones  │
│  cada vez más abstractas de los datos.                          │
│                                                                 │
│  IA ⊃ Machine Learning ⊃ Deep Learning                          │
│                                                                 │
│  ML TRADICIONAL:                                                │
│  Datos ──▶ [Feature Engineering Manual] ──▶ Modelo ──▶ Salida   │
│            (tú decides qué features)                            │
│                                                                 │
│  DEEP LEARNING:                                                 │
│  Datos ──▶ [Red Neuronal Profunda] ──▶ Salida                   │
│            (aprende features automáticamente)                   │
│                                                                 │
│  EJEMPLO EN IMÁGENES:                                           │
│                                                                 │
│  Capa 1: Detecta bordes (──, |, /, \)                           │
│      ↓                                                          │
│  Capa 2: Detecta formas (círculos, cuadrados)                   │
│      ↓                                                          │
│  Capa 3: Detecta partes (ojos, nariz, ruedas)                   │
│      ↓                                                          │
│  Capa 4: Detecta objetos (cara, carro)                          │
│                                                                 │
│  CUÁNDO USAR DL:                                                │
│  ✅ Imágenes, audio, video, texto                               │
│  ✅ MUCHOS datos (millones de ejemplos)                         │
│  ✅ Problemas complejos donde feature eng. es difícil           │
│  ✅ GPU disponible para entrenamiento                           │
│                                                                 │
│  CUÁNDO NO USAR:                                                │
│  ❌ Datos tabulares pequeños (XGBoost gana)                     │
│  ❌ Necesitas interpretabilidad                                 │
│  ❌ Pocos datos de entrenamiento                                │
│  ❌ Recursos computacionales limitados                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Perceptrón y Redes Neuronales

### El Perceptrón (Neurona Artificial)

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: PERCEPTRÓN                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  La unidad básica de una red neuronal.                          │
│  Inspirado (vagamente) en neuronas biológicas.                  │
│                                                                 │
│       x₁ ──w₁──┐                                                │
│                │                                                │
│       x₂ ──w₂──┼──▶ [Σ + b] ──▶ [f] ──▶ salida                 │
│                │      suma      función                         │
│       x₃ ──w₃──┘     ponderada  activación                      │
│                                                                 │
│  PROCESO:                                                       │
│  1. Multiplicar entradas por pesos: w₁x₁ + w₂x₂ + w₃x₃          │
│  2. Sumar bias: z = Σwᵢxᵢ + b                                   │
│  3. Aplicar función de activación: salida = f(z)                │
│                                                                 │
│  FUNCIÓN DE ACTIVACIÓN (introduce no-linealidad):               │
│                                                                 │
│  Sin activación: solo puede aprender relaciones lineales        │
│  Con activación: puede aprender CUALQUIER función               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Funciones de Activación

```
┌─────────────────────────────────────────────────────────────────┐
│              FUNCIONES DE ACTIVACIÓN                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SIGMOID: σ(z) = 1/(1+e⁻ᶻ)                                   │
│     Rango: (0, 1)                                               │
│     Uso: Capa de salida para clasificación binaria              │
│     Problema: Vanishing gradient en capas profundas             │
│                                                                 │
│  2. TANH: tanh(z) = (eᶻ - e⁻ᶻ)/(eᶻ + e⁻ᶻ)                       │
│     Rango: (-1, 1)                                              │
│     Mejor que sigmoid (centrada en 0)                           │
│     Mismo problema de vanishing gradient                        │
│                                                                 │
│  3. ReLU: f(z) = max(0, z)  ← LA MÁS USADA                      │
│     Rango: [0, ∞)                                               │
│           │     /                                               │
│           │    /                                                │
│     ──────┼───/──▶                                              │
│           │                                                     │
│     ✅ Rápida, evita vanishing gradient                         │
│     ❌ "Dying ReLU" (neuronas que se quedan en 0)               │
│                                                                 │
│  4. Leaky ReLU: f(z) = max(0.01z, z)                            │
│     Soluciona dying ReLU                                        │
│                                                                 │
│  5. SOFTMAX (para salida multiclase):                           │
│     Convierte scores en probabilidades que suman 1              │
│     softmax(zᵢ) = eᶻⁱ / Σeᶻʲ                                   │
│                                                                 │
│  GUÍA RÁPIDA:                                                   │
│  • Capas ocultas: ReLU (o Leaky ReLU)                           │
│  • Salida binaria: Sigmoid                                      │
│  • Salida multiclase: Softmax                                   │
│  • Salida regresión: Ninguna (lineal)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Red Neuronal Multicapa (MLP)

```
┌─────────────────────────────────────────────────────────────────┐
│              ESTRUCTURA DE UNA RED NEURONAL                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    CAPA         CAPA           CAPA          CAPA               │
│    ENTRADA      OCULTA 1       OCULTA 2      SALIDA             │
│                                                                 │
│      ○            ○              ○             ○                │
│                                                                 │
│      ○ ─────────▶ ○ ──────────▶ ○ ─────────▶                   │
│                                               ○                 │
│      ○ ─────────▶ ○ ──────────▶ ○ ─────────▶                   │
│                                                                 │
│      ○            ○              ○                              │
│                                                                 │
│   (features)   (aprende       (aprende      (predicción)        │
│                features       features                          │
│                 básicas)       complejas)                       │
│                                                                 │
│  TERMINOLOGÍA:                                                  │
│  • Capa de entrada: Recibe los datos (no tiene parámetros)      │
│  • Capas ocultas: Donde ocurre el aprendizaje                   │
│  • Capa de salida: Produce la predicción                        │
│  • "Profundidad": Número de capas ocultas                       │
│  • "Ancho": Número de neuronas por capa                         │
│                                                                 │
│  PARÁMETROS A APRENDER:                                         │
│  • Pesos (W): Conexiones entre neuronas                         │
│  • Bias (b): Término independiente por neurona                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# RED NEURONAL BÁSICA CON PYTORCH
# ========================================
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# Crear datos
X, y = make_classification(n_samples=1000, n_features=20, n_classes=2, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Escalar
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# Convertir a tensores de PyTorch
X_train_t = torch.FloatTensor(X_train)
y_train_t = torch.FloatTensor(y_train).unsqueeze(1)
X_test_t = torch.FloatTensor(X_test)
y_test_t = torch.FloatTensor(y_test).unsqueeze(1)

# Definir la red neuronal
class NeuralNetwork(nn.Module):
    def __init__(self, input_size):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(input_size, 64),   # Capa 1: 20 → 64
            nn.ReLU(),                    # Activación
            nn.Dropout(0.2),              # Regularización
            nn.Linear(64, 32),            # Capa 2: 64 → 32
            nn.ReLU(),
            nn.Dropout(0.2),
            nn.Linear(32, 1),             # Salida: 32 → 1
            nn.Sigmoid()                  # Probabilidad
        )
    
    def forward(self, x):
        return self.network(x)

# Crear modelo
model = NeuralNetwork(input_size=20)
print(model)

# Loss y optimizador
criterion = nn.BCELoss()  # Binary Cross Entropy
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Entrenamiento
epochs = 100
for epoch in range(epochs):
    model.train()
    
    # Forward pass
    outputs = model(X_train_t)
    loss = criterion(outputs, y_train_t)
    
    # Backward pass
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    
    if (epoch + 1) % 20 == 0:
        model.eval()
        with torch.no_grad():
            test_outputs = model(X_test_t)
            predictions = (test_outputs > 0.5).float()
            accuracy = (predictions == y_test_t).float().mean()
            print(f'Epoch [{epoch+1}/{epochs}], Loss: {loss.item():.4f}, Acc: {accuracy:.4f}')
```

---

## Backpropagation

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: BACKPROPAGATION                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  El algoritmo que permite a las redes neuronales APRENDER.      │
│  Calcula cómo ajustar cada peso para reducir el error.          │
│                                                                 │
│  PROCESO (simplificado):                                        │
│                                                                 │
│  1. FORWARD PASS (Paso hacia adelante)                          │
│     Datos ──▶ Capa 1 ──▶ Capa 2 ──▶ ... ──▶ Predicción          │
│                                                                 │
│  2. CALCULAR ERROR (Loss)                                       │
│     Error = diferencia entre predicción y valor real            │
│                                                                 │
│  3. BACKWARD PASS (Paso hacia atrás)                            │
│     Propagar el error hacia atrás usando la REGLA DE LA CADENA  │
│     Calcular: ∂Error/∂peso para cada peso                       │
│                                                                 │
│  4. ACTUALIZAR PESOS                                            │
│     peso_nuevo = peso_viejo - learning_rate × gradiente         │
│                                                                 │
│  INTUICIÓN:                                                     │
│  "¿Cuánto contribuyó cada peso al error?"                       │
│  "Si este peso es 'culpable', ajustarlo más"                    │
│                                                                 │
│  GRADIENT DESCENT:                                              │
│                                                                 │
│       Error │\                                                  │
│             │ \                                                 │
│             │  \    ← Seguir el gradiente                       │
│             │   \_      hacia abajo                             │
│             │     \__                                           │
│             │        \___●  ← Mínimo                            │
│             └──────────────── pesos                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Optimizadores

```
┌─────────────────────────────────────────────────────────────────┐
│              OPTIMIZADORES COMUNES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SGD (Stochastic Gradient Descent)                           │
│     w = w - lr × gradiente                                      │
│     Simple pero puede ser lento                                 │
│                                                                 │
│  2. SGD + Momentum                                              │
│     Añade "inercia" para acelerar y suavizar                    │
│     Como una bola rodando por una colina                        │
│                                                                 │
│  3. ADAM (Adaptive Moment Estimation) ← MÁS USADO               │
│     Combina momentum + learning rate adaptativo                 │
│     Cada parámetro tiene su propio learning rate                │
│     ✅ Funciona bien out-of-the-box                             │
│     Hiperparámetros: lr=0.001, betas=(0.9, 0.999)               │
│                                                                 │
│  4. AdamW                                                       │
│     Adam con weight decay correcto                              │
│     Mejor para modelos grandes                                  │
│                                                                 │
│  GUÍA:                                                          │
│  • Por defecto: Adam(lr=0.001)                                  │
│  • CNNs: SGD + Momentum puede funcionar mejor                   │
│  • Transformers: AdamW                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Regularización en Deep Learning

```
┌─────────────────────────────────────────────────────────────────┐
│              TÉCNICAS DE REGULARIZACIÓN                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DROPOUT                                                     │
│     Durante entrenamiento, "apaga" neuronas aleatoriamente      │
│                                                                 │
│     Sin dropout:    ○──○──○──○                                  │
│     Con dropout:    ○──✗──○──✗  (algunas apagadas)             │
│                                                                 │
│     Efecto: Fuerza a la red a no depender de neuronas           │
│     específicas. Como entrenar múltiples redes.                 │
│     Típico: dropout=0.2 a 0.5                                   │
│                                                                 │
│  2. BATCH NORMALIZATION                                         │
│     Normaliza la salida de cada capa                            │
│     ✅ Permite learning rates más altos                         │
│     ✅ Reduce dependencia de inicialización                     │
│     ✅ Efecto regularizador                                     │
│                                                                 │
│  3. EARLY STOPPING                                              │
│     Detener entrenamiento cuando val_loss deja de mejorar       │
│                                                                 │
│     Loss │\                                                     │
│          │ \  train                                             │
│          │  \_____                                              │
│          │    \      ← DETENER AQUÍ                             │
│          │     \____ val (empieza a subir)                      │
│          └──────────── epochs                                   │
│                                                                 │
│  4. DATA AUGMENTATION                                           │
│     Crear más datos artificialmente                             │
│     Imágenes: rotar, voltear, zoom, crop                        │
│     Texto: sinónimos, back-translation                          │
│                                                                 │
│  5. WEIGHT DECAY (L2)                                           │
│     Penalizar pesos grandes                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# RED CON TÉCNICAS DE REGULARIZACIÓN
# ========================================
import torch.nn as nn

class RegularizedNetwork(nn.Module):
    def __init__(self, input_size, num_classes):
        super().__init__()
        
        self.layers = nn.Sequential(
            # Capa 1
            nn.Linear(input_size, 256),
            nn.BatchNorm1d(256),      # Batch Normalization
            nn.ReLU(),
            nn.Dropout(0.3),          # Dropout 30%
            
            # Capa 2
            nn.Linear(256, 128),
            nn.BatchNorm1d(128),
            nn.ReLU(),
            nn.Dropout(0.3),
            
            # Capa 3
            nn.Linear(128, 64),
            nn.BatchNorm1d(64),
            nn.ReLU(),
            nn.Dropout(0.2),
            
            # Salida
            nn.Linear(64, num_classes)
        )
    
    def forward(self, x):
        return self.layers(x)

# Optimizador con weight decay
model = RegularizedNetwork(input_size=20, num_classes=10)
optimizer = torch.optim.AdamW(
    model.parameters(), 
    lr=0.001, 
    weight_decay=0.01  # L2 regularization
)

# Early Stopping manual
class EarlyStopping:
    def __init__(self, patience=5):
        self.patience = patience
        self.counter = 0
        self.best_loss = float('inf')
        self.early_stop = False
    
    def __call__(self, val_loss):
        if val_loss < self.best_loss:
            self.best_loss = val_loss
            self.counter = 0
        else:
            self.counter += 1
            if self.counter >= self.patience:
                self.early_stop = True
        return self.early_stop

early_stopping = EarlyStopping(patience=10)
```

---

## Redes Convolucionales (CNN)

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: REDES CONVOLUCIONALES                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Diseñadas para IMÁGENES. Explotan la estructura espacial.      │
│                                                                 │
│  ¿POR QUÉ NO USAR MLP PARA IMÁGENES?                            │
│  Imagen 224×224×3 = 150,528 pixels                              │
│  MLP: Cada neurona conectada a TODOS → millones de parámetros   │
│  CNN: Comparte pesos → muchos menos parámetros                  │
│                                                                 │
│  OPERACIÓN DE CONVOLUCIÓN:                                      │
│                                                                 │
│  Imagen          Filtro/Kernel       Feature Map                │
│  ┌─────────┐     ┌───┐                                          │
│  │1 2 3 4 5│     │1 0│               ┌───────┐                  │
│  │2 3 4 5 6│  *  │0 1│       =       │ 4  6  │                  │
│  │3 4 5 6 7│     └───┘               │ 6  8  │                  │
│  └─────────┘                         └───────┘                  │
│                                                                 │
│  El filtro "desliza" por la imagen detectando patrones          │
│  Diferentes filtros detectan diferentes características         │
│                                                                 │
│  ARQUITECTURA TÍPICA:                                           │
│                                                                 │
│  Imagen → [Conv → ReLU → Pool] × N → Flatten → FC → Salida      │
│                                                                 │
│  • Conv: Extrae características                                 │
│  • ReLU: No linealidad                                          │
│  • Pooling: Reduce dimensión, invarianza a posición             │
│  • FC (Fully Connected): Clasificación final                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│              CAPAS DE UNA CNN                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. CONVOLUCIÓN (Conv2d)                                        │
│     Parámetros:                                                 │
│     • in_channels: Canales de entrada (3 para RGB)              │
│     • out_channels: Número de filtros                           │
│     • kernel_size: Tamaño del filtro (3×3, 5×5)                 │
│     • stride: Paso del deslizamiento                            │
│     • padding: Añadir bordes para mantener tamaño               │
│                                                                 │
│  2. POOLING                                                     │
│     MaxPool: Toma el máximo de una región                       │
│     ┌───┬───┐                                                   │
│     │1 3│     → max = 4                                         │
│     │2 4│                                                       │
│     └───┴───┘                                                   │
│     AvgPool: Promedio de una región                             │
│     Efecto: Reduce tamaño, añade invarianza                     │
│                                                                 │
│  3. FLATTEN                                                     │
│     Convierte el tensor 3D en vector 1D para FC                 │
│     [batch, channels, H, W] → [batch, channels×H×W]             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# CNN PARA CLASIFICACIÓN DE IMÁGENES
# ========================================
import torch
import torch.nn as nn
import torchvision
import torchvision.transforms as transforms

# CNN para CIFAR-10 (imágenes 32×32×3, 10 clases)
class CNN(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        
        # Capas convolucionales
        self.conv_layers = nn.Sequential(
            # Conv1: 3 canales → 32 filtros
            nn.Conv2d(3, 32, kernel_size=3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),  # 32×32 → 16×16
            
            # Conv2: 32 → 64 filtros
            nn.Conv2d(32, 64, kernel_size=3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),  # 16×16 → 8×8
            
            # Conv3: 64 → 128 filtros
            nn.Conv2d(64, 128, kernel_size=3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(),
            nn.MaxPool2d(2, 2),  # 8×8 → 4×4
        )
        
        # Capas fully connected
        self.fc_layers = nn.Sequential(
            nn.Flatten(),
            nn.Linear(128 * 4 * 4, 256),
            nn.ReLU(),
            nn.Dropout(0.5),
            nn.Linear(256, num_classes)
        )
    
    def forward(self, x):
        x = self.conv_layers(x)
        x = self.fc_layers(x)
        return x

# Crear modelo
model = CNN(num_classes=10)
print(model)

# Contar parámetros
total_params = sum(p.numel() for p in model.parameters())
print(f"\nTotal parámetros: {total_params:,}")

# Data augmentation para imágenes
train_transform = transforms.Compose([
    transforms.RandomHorizontalFlip(),
    transforms.RandomRotation(10),
    transforms.RandomCrop(32, padding=4),
    transforms.ToTensor(),
    transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))
])
```

### Transfer Learning

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: TRANSFER LEARNING                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Usar un modelo pre-entrenado (en millones de imágenes)         │
│  y adaptarlo a tu problema específico.                          │
│                                                                 │
│  ¿POR QUÉ FUNCIONA?                                             │
│  Las primeras capas aprenden features genéricas (bordes,        │
│  texturas) que son útiles para cualquier imagen.                │
│                                                                 │
│  ESTRATEGIAS:                                                   │
│                                                                 │
│  1. FEATURE EXTRACTION (Pocos datos)                            │
│     Congelar todas las capas, solo entrenar clasificador        │
│                                                                 │
│     [Backbone congelado] ──▶ [Nuevo clasificador]               │
│                                                                 │
│  2. FINE-TUNING (Más datos)                                     │
│     Descongelar algunas capas finales y entrenar                │
│                                                                 │
│     [Capas congeladas] ──▶ [Capas entrenables] ──▶ [Clasificador]│
│                                                                 │
│  MODELOS PRE-ENTRENADOS POPULARES:                              │
│  • ResNet (18, 34, 50, 101, 152)                                │
│  • EfficientNet (B0-B7)                                         │
│  • VGG (16, 19)                                                 │
│  • MobileNet (para dispositivos móviles)                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# TRANSFER LEARNING CON PYTORCH
# ========================================
import torchvision.models as models

# Cargar ResNet50 pre-entrenado
model = models.resnet50(weights='IMAGENET1K_V2')

# Ver estructura
print(f"Capas originales: {model.fc}")  # Linear(2048, 1000) para ImageNet

# ESTRATEGIA 1: Feature Extraction
# Congelar todas las capas
for param in model.parameters():
    param.requires_grad = False

# Reemplazar clasificador final
num_classes = 10  # Tu número de clases
model.fc = nn.Sequential(
    nn.Linear(2048, 256),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(256, num_classes)
)

# Solo entrena el clasificador nuevo
optimizer = torch.optim.Adam(model.fc.parameters(), lr=0.001)

# ESTRATEGIA 2: Fine-tuning
# Descongelar últimas capas
for param in model.layer4.parameters():
    param.requires_grad = True

# Learning rate menor para capas pre-entrenadas
optimizer = torch.optim.Adam([
    {'params': model.layer4.parameters(), 'lr': 1e-4},
    {'params': model.fc.parameters(), 'lr': 1e-3}
])
```

---

## Redes Recurrentes (RNN)

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: REDES RECURRENTES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Diseñadas para SECUENCIAS donde el orden importa.              │
│  Tienen "memoria" de pasos anteriores.                          │
│                                                                 │
│  CASOS DE USO:                                                  │
│  • Texto (secuencia de palabras)                                │
│  • Series temporales (precios, sensores)                        │
│  • Audio (secuencia de samples)                                 │
│  • Video (secuencia de frames)                                  │
│                                                                 │
│  ESTRUCTURA:                                                    │
│                                                                 │
│       ┌──────┐     ┌──────┐     ┌──────┐                        │
│   h₀ →│ RNN  │──h₁→│ RNN  │──h₂→│ RNN  │── h₃                  │
│       └──┬───┘     └──┬───┘     └──┬───┘                        │
│          ↑            ↑            ↑                            │
│         x₁           x₂           x₃                            │
│       (Hola)       (mundo)       (!)                            │
│                                                                 │
│  hₜ = Estado oculto (la "memoria")                              │
│  hₜ = f(hₜ₋₁, xₜ)                                               │
│                                                                 │
│  PROBLEMA: Vanishing/Exploding Gradients                        │
│  En secuencias largas, los gradientes se desvanecen o explotan  │
│  La red "olvida" información de hace muchos pasos               │
│                                                                 │
│  SOLUCIÓN: LSTM y GRU                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### LSTM (Long Short-Term Memory)

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: LSTM                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Añade "gates" (puertas) para controlar qué información         │
│  recordar y qué olvidar.                                        │
│                                                                 │
│  TRES PUERTAS:                                                  │
│                                                                 │
│  1. FORGET GATE: ¿Qué olvidar del estado anterior?              │
│     "Ya no necesito recordar el sujeto anterior"                │
│                                                                 │
│  2. INPUT GATE: ¿Qué nueva información guardar?                 │
│     "Esta palabra es importante, guardarla"                     │
│                                                                 │
│  3. OUTPUT GATE: ¿Qué parte del estado usar como salida?        │
│     "Para predecir la siguiente palabra, uso esta parte"        │
│                                                                 │
│  CELL STATE (cₜ): La "memoria a largo plazo"                    │
│  Fluye a través de la red con pocas modificaciones              │
│  Permite mantener información por muchos pasos                  │
│                                                                 │
│  GRU (Gated Recurrent Unit):                                    │
│  Versión simplificada de LSTM (2 gates en vez de 3)             │
│  Similar rendimiento, menos parámetros                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# LSTM PARA CLASIFICACIÓN DE TEXTO
# ========================================
import torch
import torch.nn as nn

class LSTMClassifier(nn.Module):
    def __init__(self, vocab_size, embedding_dim, hidden_dim, num_classes, num_layers=2):
        super().__init__()
        
        # Embedding: Convierte índices de palabras a vectores
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        
        # LSTM
        self.lstm = nn.LSTM(
            input_size=embedding_dim,
            hidden_size=hidden_dim,
            num_layers=num_layers,
            batch_first=True,
            bidirectional=True,  # Lee hacia adelante Y hacia atrás
            dropout=0.3
        )
        
        # Clasificador
        self.fc = nn.Sequential(
            nn.Linear(hidden_dim * 2, 64),  # *2 por bidireccional
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(64, num_classes)
        )
    
    def forward(self, x):
        # x: [batch, seq_len] - índices de palabras
        
        # Embedding: [batch, seq_len, embedding_dim]
        embedded = self.embedding(x)
        
        # LSTM: [batch, seq_len, hidden_dim * 2]
        lstm_out, (hidden, cell) = self.lstm(embedded)
        
        # Usar el último hidden state
        # Concatenar forward y backward
        hidden_forward = hidden[-2, :, :]
        hidden_backward = hidden[-1, :, :]
        hidden_combined = torch.cat((hidden_forward, hidden_backward), dim=1)
        
        # Clasificar
        output = self.fc(hidden_combined)
        return output

# Crear modelo
model = LSTMClassifier(
    vocab_size=10000,
    embedding_dim=128,
    hidden_dim=256,
    num_classes=2  # Sentimiento positivo/negativo
)

# Ejemplo de uso
batch = torch.randint(0, 10000, (32, 50))  # 32 oraciones, 50 palabras
output = model(batch)
print(f"Output shape: {output.shape}")  # [32, 2]
```

---

## Transformers

```
┌─────────────────────────────────────────────────────────────────┐
│              CONCEPTO: TRANSFORMERS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  La arquitectura que revolucionó NLP y ahora domina DL.         │
│  Base de GPT, BERT, T5, y todos los LLMs modernos.              │
│                                                                 │
│  INNOVACIÓN CLAVE: SELF-ATTENTION                               │
│                                                                 │
│  RNN: Procesa secuencia paso a paso (lento, olvida)             │
│  Transformer: Procesa TODA la secuencia en paralelo             │
│                                                                 │
│  SELF-ATTENTION:                                                │
│  "Para entender cada palabra, mira TODAS las demás palabras"    │
│                                                                 │
│  Ejemplo: "El banco estaba lleno de gente"                      │
│  ¿"banco" = institución financiera o lugar para sentarse?       │
│  Attention mira "gente" → probablemente lugar para sentarse     │
│                                                                 │
│  Ejemplo: "El banco cerró a las 5pm"                            │
│  Attention mira "cerró", "5pm" → institución financiera         │
│                                                                 │
│  ARQUITECTURA:                                                  │
│                                                                 │
│  Input → [Positional Encoding]                                  │
│            ↓                                                    │
│        [Multi-Head Attention]                                   │
│            ↓                                                    │
│        [Add & Normalize]                                        │
│            ↓                                                    │
│        [Feed Forward]                                           │
│            ↓                                                    │
│        [Add & Normalize] → Output                               │
│                                                                 │
│  Este bloque se repite N veces (6, 12, 24, etc.)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│              SELF-ATTENTION MECANISMO                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Para cada palabra, calcula:                                    │
│  • Query (Q): "¿Qué estoy buscando?"                            │
│  • Key (K): "¿Qué información tengo?"                           │
│  • Value (V): "¿Qué información doy si me eligen?"              │
│                                                                 │
│  PROCESO:                                                       │
│  1. Calcular scores: Q × K^T (similitud entre palabras)         │
│  2. Normalizar: softmax(scores / √d)                            │
│  3. Weighted sum: scores × V                                    │
│                                                                 │
│  MULTI-HEAD ATTENTION:                                          │
│  Múltiples "cabezas" de atención en paralelo                    │
│  Cada cabeza puede capturar diferentes relaciones               │
│  Una cabeza: relación sintáctica                                │
│  Otra cabeza: relación semántica                                │
│                                                                 │
│  POSITIONAL ENCODING:                                           │
│  Transformer no tiene noción de orden (procesa en paralelo)     │
│  Se añade información de posición a cada token                  │
│  pos_encoding = sin/cos functions de la posición                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```python
# ========================================
# USAR TRANSFORMERS PRE-ENTRENADOS (HuggingFace)
# ========================================
# pip install transformers

from transformers import (
    AutoTokenizer, 
    AutoModelForSequenceClassification,
    Trainer,
    TrainingArguments
)
import torch

# Cargar BERT pre-entrenado
model_name = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(
    model_name, 
    num_labels=2  # Clasificación binaria
)

# Tokenizar texto
text = "This movie was absolutely amazing!"
inputs = tokenizer(
    text, 
    return_tensors="pt",
    padding=True,
    truncation=True,
    max_length=128
)

print(f"Input IDs: {inputs['input_ids']}")
print(f"Attention Mask: {inputs['attention_mask']}")

# Inferencia
model.eval()
with torch.no_grad():
    outputs = model(**inputs)
    predictions = torch.softmax(outputs.logits, dim=-1)
    print(f"Probabilidades: {predictions}")

# Fine-tuning con Trainer
training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    per_device_eval_batch_size=64,
    warmup_steps=500,
    weight_decay=0.01,
    logging_dir="./logs",
    evaluation_strategy="epoch"
)

# trainer = Trainer(
#     model=model,
#     args=training_args,
#     train_dataset=train_dataset,
#     eval_dataset=eval_dataset
# )
# trainer.train()
```

### Arquitecturas Modernas

```
┌─────────────────────────────────────────────────────────────────┐
│              MODELOS TRANSFORMER POPULARES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ENCODER-ONLY (entender texto):                                 │
│  • BERT: Bidireccional, clasificación, QA, NER                  │
│  • RoBERTa: BERT mejorado                                       │
│  • DistilBERT: BERT más pequeño y rápido                        │
│                                                                 │
│  DECODER-ONLY (generar texto):                                  │
│  • GPT-2, GPT-3, GPT-4: Generación de texto                     │
│  • LLaMA: Meta's open-source LLM                                │
│                                                                 │
│  ENCODER-DECODER (transformar texto):                           │
│  • T5: Text-to-Text Transfer Transformer                        │
│  • BART: Denoising autoencoder                                  │
│  • MarianMT: Traducción                                         │
│                                                                 │
│  PARA IMÁGENES:                                                 │
│  • ViT (Vision Transformer): Imágenes como secuencia de patches │
│  • CLIP: Conecta imágenes y texto                               │
│  • DALL-E: Genera imágenes desde texto                          │
│                                                                 │
│  MULTIMODALES:                                                  │
│  • GPT-4V: Texto + imágenes                                     │
│  • Gemini: Texto + imágenes + audio + video                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🏷️ Tags

#programming #deeplearning #pytorch #neuralnetworks #ai #transformers

---

## 📚 Ver También

- [[ML Fundamentals Guide|Fundamentos de ML]]
- [[ML Algorithms Guide|Algoritmos de ML]]
- [[ML Projects Guide|Proyectos Prácticos]]
