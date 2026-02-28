### Filtro Exponencial

El filtro exponencial se utiliza para suavizar señales ruidosas, como los ángulos de los motores, dando más peso al valor anterior y menos al valor actual.

**Fórmula:**  
\[
y[n] = \alpha \cdot x[n] + (1 - \alpha) \cdot y[n-1]
\]

- `x[n]` = valor actual de la señal  
- `y[n]` = valor filtrado  
- `y[n-1]` = valor filtrado anterior  
- `α` = coeficiente del filtro (0 < α ≤ 1)

**Notas:**  
- α cercano a 1 → sigue rápido la señal (menos suavizado)  
- α cercano a 0 → suaviza más la señal (más lento para reaccionar)  

En este proyecto, se aplica a los ángulos de los motores para obtener valores más estables y evitar oscilaciones bruscas.
