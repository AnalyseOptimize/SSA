# Введение
Метод SSA (Singular spectrum analysis) используется для анализа временных рядов через траекторную матрицу и SVD разложение. Подробнее о методе можно почитать в Golyandina N., Nekrutkin V., Zhigljavsky A. *Analysis of Time Series Structure, 2001* или в файле `Presentation SSA.pdf`. Примеры применения можно найти в *Вохмянин С.В., 2010*, *Щигрев С.В., 2008*. 

Этот репозиторий использовался для исследования SSA в рамках курсовой работы.
# Навигация
- `models` содержит имлементацию SSA и LRF (linear recurrent formula).
- `Approximation research` - результаты по разложению ряда частями. Подробнее - в основной работе `paper.pdf`
- `data` - все датасеты, использованные для анализа
- `Overview` - все вспомогательные файлы и графики для иллюстрации работы метода
- `Introduction to SSA.ipynb`, `Theory overview.ipynb`, `SSA implementation.ipynb` - иллюстративные ноутбуки с простым кодом для реализации метода
- `paper.pdf` - полный обзор метода, полученных результатов, возможных расширений. Обзор проводился для 3 задач: сглаживания, прогнозирования и выделения компонент ряда.

# Singular Spectrum Analysis

Пусть дан временной ряд $f = (f_1, \dots, f_n)$. Поставим задачу аддитивно разложить временной ряд на три классические компоненты: тренд, сезонность и шум, - т.е. представить его в виде $f = g_t + g_s + \epsilon$, где $g_t$ - трендовая компонента, $g_s$ - сезонная, $\epsilon$ - шум. Шум стоит понимать как необъясненную ошибку аппроксимации, а не стохастическую компоненту. В задаче сглаживания необходимо выделить только первые две компоненты (сигнал), очистив ряд от шума.

Идея метода SSA (_Singular Spectrum Analysis_) основана на малоранговом приближении скользящих векторов фиксированной длины и состоит из 4 этапов.

__1 этап: составление траекторной матрицы__

Для фиксированной длины окна $l \in \mathbb{N}$ составим матрицу $S:$

$$
S = 
\begin{pmatrix}
  f_1& f_2 & \cdots & f_{n-l+1}\\
  f_2& f_3 & \cdots & f_{n-l+2} \\
  \vdots & & \vdots & \vdots \\
  f_l & f_{l+1} & \cdots & f_{n}
\end{pmatrix}
$$

Матрица $S$ называется траекторной матрицей ряда и имеет размерность $\mathbb{R}^{l\times (n-l+1)}$. $S$ - ганкелева матрица, т.е. на каждой её побочной диагонали стоит одинаковое число. Соответственно, если переставить столбцы матрицы в обратном порядке, то получится Тёплицева матрица (матрица, в которой на диагоналях стоят одинаковые элементы). 

__2 этап: SVD разложение__

Траекторная матрица представляется в виде $S = U\Sigma V^T$ (_Singular Value Decomposition_), где $U,V$ - ортогональные матрицы, а $\Sigma$ - диагональная. В силу диагональности разложение можно переписать в виде суммы и ограничить его ранг: $S \approx \sum_{i=1}^r \sigma_i u_i v_i^T$. Здесь $u_i$ - вектор-столбец, а $v_i^T$ - вектор-строка, $r$ - ранг SVD разложения. 

По теореме Эккарта-Янга малоранговое SVD разложение является оптимальным с точки зрения минимизации Фробениусовой нормы $\| S - S' \|_F$, где $rank(S') \leq r$. Свойство оптимальности позволяет получить компоненты, которые хорошо приближают исходный ряд за счет выделения доминирующих паттернов.  

__3 этап: группировка__

После получения суммы из $r$ компонент их необходимо сгруппировать в три глобальные компоненты: $g_t, g_s, \epsilon$. Пусть $I_1, I_2, I_3$ - множество индексов, относящихся к тренду, сезонности и шуму соответственно. Тогда, $T = \sum_{i\in I_1} \sigma_i u_i v_i^T, C = \sum_{i\in I_2} \sigma_i u_i v_i^T, E = \sum_{i\in I_3}\sigma_i u_i v_i^T$.  

__4 этап: генкелизация__

Для перехода от матричной формы к ряду нужно усреднить все побочные диагонали матриц $T,C,E$ и получить искомое разложение $g_t, g_s, \epsilon$. Операция усреднения по побочным диагоналям называется генкелизацией и обладает _свойством оптимальности_ : траекторная матрица $\hat{S}$ ряда, полученного из некоторой матрицы $Z$ (не обязательно ганкелевой) диагональным усреднением, является ближайшей к $Z$ по норме Фробениуса из всех ганкелевых. 

# Документация

`SSA(l: длина окна,
  rank: ранг SVD разложения
  chunk_size: размер каждого чанка для обработки длинного ряда по частям,
  intersectrion_rate: в пересечении чанков будет участвовать intersection_rate * l наблюдений 
  )`

  ## Методы
  `.transform_to_matrix(self, series)` - возвращает трехмерный тензор: траекторные матрицы для каждой компоненты в SVD разложения (т.е. до генкелезации). На текущее время не поддерживает обработку ряда по частям.

  `.transform_to_series(self, series)` - возвращает матрицу _(n_components, series_len)_ - в строках записаны компоненты исходного ряда, т.е. после генкелезации, той же длины. Возможна обработка по частям.

  `.fit(series)` - обучает коэффициенты для предсказания
  
  `.predict(horizon)` - предсказывает по линейной рекуррентной формуле вперед на _horizon_ шагов.

  `w_corr(series, l)` - для матрицы _(num_series, series_len)_ считает w_correlation, предварительно проводя z-стандартизацию всех рядов. Подробнее о w_correlation см. `paper.pdf`.
