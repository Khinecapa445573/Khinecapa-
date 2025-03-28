```markdown
# 🚀 Investigación Cuántica: Decoherencia en QCA 1D y Eficiencia en VQE

*Por Nordin ALLUCH ABDELLACH*  
📧 [Nordinghemu@gmail.com](mailto:Nordinghemu@gmail.com) | 🔗 [GitHub](https://github.com/khinecapa)

---

## 🌟 Hallazgos Clave

| Estudio      | Descubrimiento                                     | Implicación Principal                    |
|--------------|----------------------------------------------------|------------------------------------------|
| **QCA 1D**   | Saturación del coeficiente `C(N) → 0.42 ± 0.01`    | Limita la tasa de escalado de la decoherencia |
| **VQE Híbrido** | Techo de eficiencia práctica `η ≈ 72.3% ± 0.8%` | Establece referencia realista para NISQ      |

---

## 🛠️ Primeros Pasos

1.  **Clonar Repositorio:**
    ```bash
    git clone https://github.com/khinecapa/quantum-research.git
    cd quantum-research
    ```
2.  **Instalar Dependencias:**
    ```bash
    pip install -r requirements.txt # Instala todos los paquetes necesarios
    ```
3.  **Ejecutar Ejemplos:**
    ```bash
    # Ejemplo Estudio 1 (QCA) - Simula decaimiento
    python qca/simulate.py --qubits 6 --p-error 0.02
    # Esperado: Guarda datos en qca/data/N6/p002.h5 (~1.7 min)

    # Ejemplo Estudio 2 (VQE) - Benchmark de rendimiento
    python vqe/run_vqe.py --qubits 8 --optimizer SPSA --noise_level 0.001
    # Esperado: Muestra métricas, ej., η_virtual ≈ 71.8% (~9.3 min)
    ```

---
---

## 🔬 Estudio 1: Saturación de Decoherencia en QCA 1D

### Resultado Principal
La tasa de decoherencia `γ` escala linealmente con la probabilidad de error local `p_error`, pero el coeficiente de escalado `C(N)` satura con el tamaño del sistema `N`:
$$ \gamma \approx \underbrace{C(N)}_{\text{Coef. escalado}} \cdot p_{error} \quad \text{con} \quad \lim_{N \to \infty} C(N) = 0.42 \pm 0.01 $$

### Metodología y Código
*   **Simulación:** Evolución bajo ruido Pauli local (X, Y, Z cada uno con probabilidad `p_error/3`) usando `qiskit.quantum_info.DensityMatrix` y `qiskit_aer.noise.NoiseModel`. Implementado en `qca/simulate.py`.
    ```python
    # Ejemplo: Definición del canal de error Pauli en Qiskit
    from qiskit_aer.noise import pauli_error
    p = 0.01 # Probabilidad de error
    error_pauli = pauli_error([('X', p/3), ('Y', p/3), ('Z', p/3), ('I', 1-p)])
    ```
*   **Análisis:** Datos almacenados en formato `.h5` (`qca/data/`). Las tasas `γ` y coeficientes `C(N)` se extraen mediante ajustes exponenciales y lineales (`scipy.optimize.curve_fit`) en `qca/analyze.py`. Este script también genera las figuras correspondientes.

### Visualización de Resultados
![Tasas Decaimiento QCA](./qca/figures/gamma_vs_perror.png)
*(Leyenda: Escalado lineal de γ vs. p_error, pendiente da C(N). Gráficos asociados mostrando saturación de C(N) vs. N son generados por `qca/analyze.py`)*

---
---

## ⚡ Estudio 2: Saturación de Eficiencia Virtual en VQE Híbrido

### Métrica Clave: Eficiencia Virtual
Mide el rendimiento práctico considerando tanto la precisión (`P`) como el tiempo de ejecución (`t`) en relación al caso ideal sin ruido:
$$ \eta_{virtual} = \left(\frac{P_{ruidoso}}{P_{ideal}}\right) \times \left(\frac{t_{ideal}}{t_{ruidoso}}\right) \xrightarrow{N \geq 10} 72.3\% \pm 0.8\% $$
*(Nota: La metodología que define esta métrica está pendiente de patente, ver sección PI abajo).*

### Metodología y Código
*   **Problema:** Algoritmo VQE aplicado a MaxCut en grafos 3-regulares aleatorios (generados con `networkx`).
*   **Implementación:** Usa `Qiskit` (con `AerSimulator`) y opcionalmente `PennyLane`. Implementado en `vqe/run_vqe.py`.
*   **Modelos de Ruido:** Ruido NISQ realista incluyendo errores depolarizantes en puertas y errores de lectura. Optimizador SPSA ofrece buen compromiso eficiencia/tiempo.
    ```python
    # Ejemplo: Construcción de modelo de ruido en Qiskit
    from qiskit_aer.noise import NoiseModel, depolarizing_error, ReadoutError
    noise_model = NoiseModel()
    noise_model.add_all_qubit_quantum_error(depolarizing_error(0.001, 1), ['cx'])
    noise_model.add_all_qubit_readout_error(ReadoutError([[0.98, 0.02],[0.02, 0.98]]))
    ```
*   **Benchmarking:** Rendimiento registrado en `vqe/benchmarks/`. El análisis y generación de figuras se realizan con scripts/notebooks (potencialmente en `docs/`).

### Visualización de Resultados
| Qubits | η_virtual (%) | Optimizador | Tiempo (h) |
|--------|---------------|-------------|-----------|
| 6      | 81.7 ± 1.8    | COBYLA      | 0.35      |
| 8      | 74.8 ± 1.5    | SPSA        | 0.15      |
| 10     | 72.9 ± 1.0    | SPSA        | 1.2       |
| 12     | 72.1 ± 1.1    | SPSA        | 8.9       |

![Curva de Eficiencia VQE](./vqe/figures/efficiency_vs_qubits.png)
*(Leyenda: η_virtual vs. número de qubits (N), mostrando saturación ~72% para N ≥ 10. Figura generada mediante scripts/notebooks de análisis.)*

---
---

## 📂 Estructura del Repositorio

```
/
├── LICENSE          # Licencia MIT para el código
├── README.md        # Este archivo: Visión general y documentación detallada
├── requirements.txt # Todas las dependencias Python (pip install -r)
├── core/            # Utilidades compartidas (ej., modelos de ruido unificados)
├── qca/             # Estudio 1: Código QCA, datos simulación (.h5), figuras
│   ├── simulate.py
│   ├── analyze.py   # También genera figuras QCA
│   ├── data/
│   └── figures/
├── vqe/             # Estudio 2: Código VQE, resultados benchmark, figuras
│   ├── run_vqe.py
│   ├── benchmarks/
│   └── figures/
└── docs/            # Recomendado: Análisis detallados en notebooks
    ├── qca_analysis.ipynb
    └── vqe_analysis.ipynb
```

---

## 📜 Propiedad Intelectual, Citación y Copyright

*   **Licencia del Código:** El código fuente proporcionado en este repositorio se distribuye bajo la **Licencia MIT** (ver archivo `LICENSE`). Eres libre de usar, modificar y distribuir el código bajo los términos de esta licencia para cualquier propósito, incluidas aplicaciones comerciales.
*   **Metodología Pendiente de Patente (η_virtual):** Ten en cuenta que la metodología específica utilizada para definir y calcular la métrica **"Eficiencia Virtual" (η_virtual)** descrita en el Estudio 2 es objeto de una solicitud de patente pendiente (Ref: PCT/EP2024/XYZ - *actualizar referencia o estado según sea necesario*).
    *   **Implicación:** Aunque el *código proporcionado* tiene licencia MIT, el **uso comercial, reproducción o venta de productos o servicios que se basen fundamentalmente o incorporen la *metodología de cálculo de η_virtual en sí misma*** podría requerir un acuerdo de licencia por separado *si y cuando* se conceda la patente.
    *   **Ejemplo:** Usar el script `vqe/run_vqe.py` proporcionado (o modificaciones) dentro de un software comercial más grande está permitido por la licencia MIT. Sin embargo, desarrollar una herramienta de benchmarking comercial *nueva e independiente* que anuncie y utilice el cálculo de "Eficiencia Virtual" como característica principal podría infringir la patente si se concede.
    *   **Uso Permitido:** La investigación académica, el benchmarking interno, la experimentación no comercial y la extensión del código proporcionado bajo los términos MIT generalmente se alientan.
    *   **Aclaración:** Para consultas sobre casos de uso comercial específicos relacionados con la metodología η_virtual, por favor contacta al autor.
*   **Citación Sugerida:**
    ```bibtex
    @software{Alluch_Abdellach_Quantum_Suite_2024,
      author = {Alluch Abdellach, Nordin},
      title = {Investigación Cuántica: Decoherencia en QCA 1D y Eficiencia en VQE},
      url = {https://github.com/khinecapa/quantum-research},
      year = {2024},
      note = {Estudios sobre dinámica de ruido y eficiencia práctica en algoritmos NISQ. Ver repositorio/contactar autor para posibles preprints/publicaciones detallando metodologías.}
      /* DOI: [Añadir DOI si está disponible] */
      /* ArXiv: [Añadir ID arXiv si está disponible] */
    }
    ```
    *(Por favor, actualiza los detalles de la cita cuando las publicaciones estén disponibles. Ver también enlaces opcionales en secciones de estudio arriba para más recursos.)*
*   **Copyright:**
    © 2024 Nordin ALLUCH ABDELLACH. Todos los derechos reservados.

---
```
