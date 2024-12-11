"""
pip install pandas scikit-learn (bash)
Importante para activar el archivo
"""

import tkinter as tk
from tkinter import messagebox
import pandas as pd
from sklearn.preprocessing import LabelEncoder

# Cargar datos de películas
try:
    peliculas_df = pd.read_csv("peliculas.csv")
except FileNotFoundError:
    messagebox.showerror("Error", "El archivo 'peliculas.csv' no se encontró.")
    exit()

# Codificar las características categóricas (como género y clasificación) en números
label_encoder = LabelEncoder()
peliculas_df['genero_encoded'] = label_encoder.fit_transform(peliculas_df['genero'])
peliculas_df['clasificacion_encoded'] = label_encoder.fit_transform(peliculas_df['clasificacion'])

# Función para recomendar películas
def recomendar_peliculas():
    try:
        # Obtener las preferencias del usuario desde la interfaz
        genero_usuario = genero_entry.get().strip()
        clasificacion_usuario = clasificacion_entry.get().strip()
        duracion_minima = int(duracion_entry.get())
        
        if duracion_minima <= 0:
            raise ValueError("La duración mínima debe ser un número mayor que 0.")
        
        # Filtrar las películas según la duración
        peliculas_filtradas = peliculas_df[peliculas_df['duracion'] >= duracion_minima]
        
        # Filtrar las películas según género y clasificación (en minúsculas para hacer la comparación insensible a mayúsculas)
        peliculas_filtradas = peliculas_filtradas[
            (peliculas_filtradas['genero'].str.lower() == genero_usuario.lower()) & 
            (peliculas_filtradas['clasificacion'].str.lower() == clasificacion_usuario.lower())
        ]
        
        # Si no hay películas que coincidan, mostrar un mensaje
        if peliculas_filtradas.empty:
            messagebox.showinfo("Recomendación", "No se encontraron películas que coincidan con tus preferencias.")
            return
        
        # Mostrar las películas recomendadas
        recomendadas = peliculas_filtradas[['titulo', 'anio', 'genero', 'duracion', 'clasificacion']]
        recomendadas_list.delete(0, tk.END)  # Limpiar la lista de recomendaciones
        
        for index, row in recomendadas.iterrows():
            recomendadas_list.insert(tk.END, f"{row['titulo']} ({row['anio']}) - {row['genero']}, {row['clasificacion']}")
    
    except ValueError as e:
        messagebox.showerror("Error", f"Por favor, ingresa un valor válido. {str(e)}")
        
# Intenta leer el archivo con una codificación diferente
try:
    peliculas_df = pd.read_csv("peliculas.csv", encoding="ISO-8859-1")
except UnicodeDecodeError:
    print("Error de codificación al intentar leer el archivo.")


# Crear la interfaz gráfica
root = tk.Tk()
root.title("Recomendador de Películas")

# Etiquetas y campos de entrada
tk.Label(root, text="Género (Ej. Drama, Acción, Ciencia ficción):").pack(pady=5)
genero_entry = tk.Entry(root)
genero_entry.pack(pady=5)

tk.Label(root, text="Clasificación (Ej. PG-13, R):").pack(pady=5)
clasificacion_entry = tk.Entry(root)
clasificacion_entry.pack(pady=5)

tk.Label(root, text="Duración mínima (en minutos):").pack(pady=5)
duracion_entry = tk.Entry(root)
duracion_entry.pack(pady=5)

# Botón para realizar la recomendación
recomendar_button = tk.Button(root, text="Recomendar Películas", command=recomendar_peliculas)
recomendar_button.pack(pady=20)

# Lista para mostrar las recomendaciones
recomendadas_list = tk.Listbox(root, width=50, height=10)
recomendadas_list.pack(pady=10)

# Iniciar la interfaz
root.mainloop()
