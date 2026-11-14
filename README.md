<div align="center">

# 🏆 ELO-Rating

### A lightweight, dependency-free ELO rating library for Java, with support for score-based rating updates

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen?style=flat-square)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-blue?style=flat-square)
![Maintained](https://img.shields.io/badge/maintained-yes-success?style=flat-square)

[Overview](#-overview) · [Features](#-features) · [Quick Start](#-quick-start) · [API](#-api-reference) · [How It Works](#-how-it-works) · [Contributing](#-contributing)

</div>

---

## 📖 Overview

The ELO rating system calculates the **relative skill levels** of players as numeric values. After every match, both players' ratings are updated based on the result **and** on how likely each player was to win beforehand.

Beating a much stronger opponent earns far more rating than beating a much weaker one. That is what makes ELO fair, and it is what this library implements.

This project goes one step beyond the original system: it can also factor in **points scored**, so a dominant 5-1 win is rewarded more than a narrow 2-1 win.

## ✨ Features

- ♟️ **Classic ELO:** updates ratings from the match outcome (win, loss, or draw)
- 🎯 **Score-aware updates:** three selectable methods that account for points scored
- ⚙️ **Fully configurable:** tune the K factor, the scale constant `c`, and the bonus factor `L` per league or competition
- 🐍☕ **Python and Java:** identical API in both languages
- 📦 **Zero dependencies:** no external libraries in either implementation

## 🚀 Quick Start

### Python

```python
from elo import ELO  # adjust the import to match your file layout

rating = ELO(k_factor=32, c_value=400, l_factor=16)

# Outcome only: 0 = draw, 1 = player A won, 2 = player B won
new_a, new_b = rating.elo(1400.0, 1550.0, 1)
print(new_a, new_b)  # ≈ 1422.5, 1527.5

# With points scored (method 2 is the default)
new_a, new_b = rating.elo_with_points(1400.0, 1550.0, 5, 1, method=2)
```

### Java

```java
ELO rating = new ELO(32, 400, 16); // adjust to match your package layout

// Outcome only: 0 = draw, 1 = player A won, 2 = player B won
double[] result = rating.elo(1400.0, 1550.0, 1);

// With points scored
double[] scored = rating.elo_with_points(1400.0, 1550.0, 5, 1, 2);
```

> [!NOTE]
> Class and package names in these snippets are illustrative. Adjust the imports to match where the `ELO` class lives in your project.

## 📚 API Reference

### `ELO` constructor

Creates a rating calculator. Create one instance per league, arena, or competition if they use different settings.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `k_factor` | int | `32` | Scaling factor `K`: how much a single match can move a rating |
| `c_value` | int | `400` | Scale constant `c` used in the expected-score formula |
| `l_factor` | int | `16` | Second scaling factor `L`: bonus weight for points scored (method 2) |

### `elo(rating_a, rating_b, outcome)`

Updates ratings using **only the match outcome**.

| Parameter | Type | Description |
|---|---|---|
| `rating_a` | float | Current rating of player A |
| `rating_b` | float | Current rating of player B |
| `outcome` | int | `0` = draw · `1` = A won · `2` = B won |

**Returns:** the new ratings as a tuple/array of floats, e.g. `(1400.0, 1550.0)`.

### `elo_with_points(rating_a, rating_b, points_a, points_b, method=2)`

Updates ratings while **also considering the points scored** by each player.

| Parameter | Type | Description |
|---|---|---|
| `rating_a` | float | Current rating of player A |
| `rating_b` | float | Current rating of player B |
| `points_a` | float | Points scored by player A |
| `points_b` | float | Points scored by player B |
| `method` | int | `0` = outcome only · `1` = points fraction replaces `S_a` · `2` = bonus using `L` (default) |

**Returns:** the new ratings as a tuple/array of floats.

## 🧮 How It Works

### Core formula

$$
R^{'}_a = R_a + K \times (S_a - E_a)
$$

| Symbol | Meaning |
|---|---|
| $R^{'}_a$ | New rating of player A |
| $R_a$ | Original rating of player A |
| $K$ | Scaling factor (generally `32`) |
| $S_a$ | Actual score: `1` for a win, `0.5` for a draw, `0` for a loss |
| $E_a$ | Expected score: the probability that player A wins |

### Expected score

The expected score is computed from the two ratings:

$$
E_a = \frac{Q_a}{Q_a + Q_b}, \quad Q_a = 10^{R_a / c}, \quad Q_b = 10^{R_b / c}, \quad c = 400
$$

Which simplifies to:

$$
E_a = \frac{1}{1 + 10^{\frac{R_b - R_a}{c}}}
$$

Since $0 \leq S_a \leq 1$ and $0 \leq E_a \leq 1$, the maximum rating change in a single match is $K$.

### Initial rating

A new player typically starts at **1000**. The starting rating should be much larger than $K$, so a new player can lose several matches before winning without the rating collapsing.

### Why consider points scored?

When the margin of victory matters, outcome-only ELO treats very different wins the same:

- Game 1: player A beats player B **5-1**
- Game 2: player C beats player B **2-1**

Player A had a far more dominant win, so should gain more rating than player C. The two score-aware methods below handle this.

### Scoring methods

| Method | Behavior | Trade-off |
|---|---|---|
| **0** | Outcome only (original ELO) | Ignores margin of victory |
| **1** | Replaces $S_a$ with the fraction of points scored | Intuitive and needs no formula change, but offers less control and predictability |
| **2** | Keeps $S_a$ as the outcome and adds a points-based bonus | Less intuitive, but gives much more control and predictability |

**Method 1**

$$
S_a = \frac{P_a}{P_a + P_b}
$$

**Method 2**

$$
R^{'}_a = R_a + K \times (S_a - E_a) + p \times L \times \left( \frac{P_a}{P_a + P_b} \right)
$$

where $P_a$ and $P_b$ are the points scored by each player, $L$ is the second scaling factor (default `16`), and:

$$
p =
\begin{cases}
  1 & \text{if } S_a - E_a > 0 \\
  -1 & \text{if } S_a - E_a < 0 \\
  0 & \text{otherwise}
\end{cases}
$$

With method 2, the maximum rating change per match is $K + L$.

## 🗂️ Dependencies

| Language | Dependencies |
|---|---|
| Python | None |
| Java | None |

## 🤝 Contributing

Contributions are welcome and appreciated.

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add your feature"`
4. Push the branch: `git push origin feature/your-feature`
5. Open a Pull Request

Good first contributions: unit tests, additional language ports, packaging for PyPI or Maven, and usage examples.

When changing behavior, keep the **Python and Java implementations in sync**.

## 🐛 Reporting Issues

Found a bug or have a suggestion? Open an issue and include the inputs you used (ratings, outcome or points, and method) and the output you expected.

## 📄 License

Add a `LICENSE` file and update this section with your chosen license. Open-source projects without a license are not legally reusable by others. [choosealicense.com](https://choosealicense.com) can help you pick one.
