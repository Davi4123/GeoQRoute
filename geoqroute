import streamlit as st
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import linprog

# ==============================
# Configuración del grid
GRID_SIZE = 10
MAX_CELLS = 50

COORDINATES = {
    'lat_min': 12.0,
    'lat_max': 12.5,
    'lon_min': -72.0,
    'lon_max': -71.5
}

# Mapa de prioridad geológica
GEO_PRIORITY = np.array([
    [0, 0, 1, 1, 2, 2, 3, 3, 0, 0],
    [0, 0, 1, 1, 2, 3, 3, 2, 0, 0],
    [0, 1, 1, 2, 3, 3, 2, 1, 0, 0],
    [0, 1, 2, 3, 3, 2, 1, 1, 0, 0],
    [1, 2, 3, 3, 2, 1, 0, 0, 0, 0],
    [1, 2, 3, 3, 2, 1, 0, 0, 0, 0],
    [0, 1, 2, 3, 3, 2, 1, 0, 0, 0],
    [0, 0, 1, 2, 3, 3, 2, 1, 0, 0],
    [0, 0, 0, 1, 2, 3, 3, 2, 0, 0],
    [0, 0, 0, 0, 1, 2, 3, 3, 1, 0]
])

# Mapa de riesgo generado aleatoriamente
np.random.seed(42)
RISK_LEVEL = np.random.choice([0, 1, 2], size=(GRID_SIZE, GRID_SIZE), p=[0.6, 0.3, 0.1])

# ==============================
# Optimización clásica
def optimize_selection(priority, risk, max_cells, w_priority=5, w_risk=3):
    n = GRID_SIZE * GRID_SIZE
    c = -1 * (priority.flatten() * w_priority - risk.flatten() * w_risk)  # maximizar prioridad, minimizar riesgo
    A = [np.ones(n)]
    b = [max_cells]
    bounds = [(0, 1) for _ in range(n)]

    result = linprog(c, A_ub=A, b_ub=b, bounds=bounds, method='highs')

    if result.success:
        x = np.round(result.x).astype(int)
        return x.reshape((GRID_SIZE, GRID_SIZE))
    else:
        st.error("Optimización fallida.")
        return np.zeros((GRID_SIZE, GRID_SIZE))

# ==============================
# Streamlit UI
st.title("🛰️ GeoQRoute")

# Create two columns with different widths
#col1, col2 = st.columns([1, 10])

# Add image in the first column

color_uniagraria = "#006B3F"  # Verde del logo UNIAGRARIA
color_text = "#FFFFFF"       # Blanco para el texto
color_slider = "#BFFF00"     # Verde limón para los sliders
color_main_bg = "#BFFF00"    # Verde limón para el fondo principal

st.markdown(f""" 
<style>
    /* Estilo para toda la página - ahora solo afecta a elementos comunes */
    .stApp {{  
        background-color: {color_main_bg};
    }}
    
    /* Estilo para el contenido principal (texto, títulos, etc.) */
    .stApp h1, .stApp h2, .stApp h3, .stApp p, .stApp li {{  
        color: #000000;
    }}
    
    /* Mantener los estilos de la barra lateral */
    .sidebar .sidebar-content {{  
        background-color: {color_uniagraria};
    }}

    .sidebar .sidebar-content * {{  
        color: {color_text};
    }}

    .stSlider > div > div > div {{  
        background-color: {color_slider} !important;
    }}
    
    /* Estilo para los bloques de código y estadísticas */
    code {{  
        background-color: rgba(0, 0, 0, 0.1);
        color: #000000;
    }}
    
    /* Asegurar que los gráficos tengan buen contraste */
    .stPlotlyChart, .stPlot {{  
        background-color: white;
        border-radius: 5px;
        padding: 10px;
    }}
</style>
""", unsafe_allow_html= True)

st.sidebar.image(
    "https://www.uniagraria.edu.co/wp-content/uploads/2018/10/uniagraria.jpg",
    width=200,
    use_container_width=False
)
st.sidebar.header("Parámetros de Optimización")
max_cells = st.sidebar.slider("Máximo de celdas seleccionadas", 10, 100, MAX_CELLS)
w_priority = st.sidebar.slider("Peso de prioridad geológica", 1, 10, 5)
w_risk = st.sidebar.slider("Peso de riesgo ambiental", 1, 10, 3)

selected_cells = optimize_selection(GEO_PRIORITY, RISK_LEVEL, max_cells, w_priority, w_risk)

# ==============================
# Visualización
fig, axs = plt.subplots(1, 3, figsize=(15, 5))

# Mapa de prioridad con flechas (similar a la imagen compartida)
im0 = axs[0].imshow(GEO_PRIORITY, cmap='Blues')
axs[0].set_title("Prioridad Geológica")

# Agregar valores numéricos en cada celda
for i in range(len(GEO_PRIORITY)):
    for j in range(len(GEO_PRIORITY[0])):
        axs[0].text(j, i, GEO_PRIORITY[i, j], ha="center", va="center", color="black")

# Agregar flechas para mostrar dirección de flujo (como en la imagen)
for i in range(1, len(GEO_PRIORITY)-1):
    for j in range(1, len(GEO_PRIORITY[0])-1):
        if GEO_PRIORITY[i, j] > 0:
            # Determinar dirección de la flecha basada en valores vecinos
            dx = 0
            dy = 0
            
            # Lógica simple para determinar dirección (hacia valores más altos)
            neighbors = [
                (i-1, j, GEO_PRIORITY[i-1, j]),  # arriba
                (i+1, j, GEO_PRIORITY[i+1, j]),  # abajo
                (i, j-1, GEO_PRIORITY[i, j-1]),  # izquierda
                (i, j+1, GEO_PRIORITY[i, j+1])   # derecha
            ]
            
            max_val = 0
            max_pos = (i, j)
            
            for ni, nj, val in neighbors:
                if val > max_val:
                    max_val = val
                    max_pos = (ni, nj)
            
            if max_val > GEO_PRIORITY[i, j]:
                dy = max_pos[0] - i
                dx = max_pos[1] - j
                axs[0].arrow(j, i, dx*0.4, dy*0.4, head_width=0.15, head_length=0.15, fc='red', ec='red')

plt.colorbar(im0, ax=axs[0])

# Mapa de riesgo
im1 = axs[1].imshow(RISK_LEVEL, cmap='Reds')
axs[1].set_title("Riesgo Ambiental")
plt.colorbar(im1, ax=axs[1])

# Celdas seleccionadas
cmap_sel = plt.cm.get_cmap('Greens', 2)
im2 = axs[2].imshow(selected_cells, cmap=cmap_sel, vmin=0, vmax=1)
axs[2].set_title("Celdas Seleccionadas")
plt.colorbar(im2, ax=axs[2], ticks=[0, 1])

# Estadísticas
num_sel = int(np.sum(selected_cells))
altas = int(np.sum((selected_cells == 1) & (GEO_PRIORITY >= 2)))
riesgo_alto = int(np.sum((selected_cells == 1) & (RISK_LEVEL >= 2)))
cobertura = np.mean(GEO_PRIORITY[selected_cells == 1] >= 2) * 100 if num_sel > 0 else 0

st.pyplot(fig)

st.markdown(f"""
**📊 Estadísticas:**
- Total celdas seleccionadas: `{num_sel}`
- Celdas de alta prioridad (≥2): `{altas}`
- Celdas con alto riesgo (≥2): `{riesgo_alto}`
- Cobertura de áreas críticas: `{cobertura:.1f}%`
""")
