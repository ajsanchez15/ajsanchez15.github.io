---
title: 'Post Template: Images, Code, Plots, and Equations'
date: 2026-03-26
permalink: /posts/2026/01/01/post-template
tags:
  - template
---

This file is a reference template showing how to include images, code blocks,
interactive plots, and math equations in a Jekyll post.

---

## Images

Place image files in the `images/` folder at the root of the repo, then
reference them with a leading `/`.

**Basic image:**

![Sample image](/images/sample-image.png)

**Image with a caption** (italic text directly below):

![Sample image](/images/sample-image.png)
*Figure 1: A sample caption describing the figure.*

**Image with controlled width** (using HTML):

<figure>
  <img src="/images/sample-image.png" alt="Sample image" style="width:60%; display:block; margin:auto;">
  <figcaption style="text-align:center;">Figure 2: Caption with centered, width-controlled image.</figcaption>
</figure>

---

## Code Blocks

Use triple backticks followed by the language name for syntax highlighting.
Supported languages include `python`, `matlab`, `tcl`, `cpp`, `bash`, and more.

**Python:**

```python
import numpy as np
import matplotlib.pyplot as plt

# Generate a simple sine wave
t = np.linspace(0, 2 * np.pi, 500)
y = np.sin(t)

plt.plot(t, y, label='sin(t)')
plt.xlabel('Time (s)')
plt.ylabel('Amplitude')
plt.title('Sine Wave')
plt.legend()
plt.savefig('images/sine-wave.png', dpi=150)
plt.show()
```

**OpenSeesPy (structural model example):**

```python
import openseespy.opensees as ops

ops.wipe()
ops.model('basic', '-ndm', 2, '-ndf', 3)

# Nodes: base (fixed) and top (free)
ops.node(1, 0.0, 0.0)
ops.node(2, 0.0, 144.0)   # 144 in = 12 ft column height

ops.fix(1, 1, 1, 1)

# Elastic column element
ops.geomTransf('Linear', 1)
ops.element('elasticBeamColumn', 1, 1, 2, 49.09, 29000, 1000, 1)
```

**MATLAB:**

```matlab
% Cyclic pushover plot
drift = linspace(-0.05, 0.05, 200);
force = 200 * tanh(50 * drift);

figure;
plot(drift * 100, force, 'b-', 'LineWidth', 1.5);
xlabel('Drift Ratio (%)');
ylabel('Lateral Force (kN)');
title('Cyclic Hysteresis Loop');
grid on;
```

**Bash / terminal commands:**

```bash
bundle exec jekyll serve --livereload
```

---

## Interactive Plots (Plotly)

Export a Plotly figure as a self-contained HTML file and store it in
`assets/plots/`. Then embed it with an `<iframe>`.

**Python export:**

```python
import plotly.graph_objects as go
import numpy as np

drift = np.linspace(-0.05, 0.05, 300)
force = 200 * np.tanh(50 * drift)

fig = go.Figure()
fig.add_trace(go.Scatter(x=drift * 100, y=force, mode='lines', name='Column'))
fig.update_layout(
    xaxis_title='Drift Ratio (%)',
    yaxis_title='Lateral Force (kN)',
    title='Interactive Hysteresis Loop'
)
fig.write_html('assets/plots/hysteresis.html')
```

**Embed in post:**

```html
<iframe src="/assets/plots/hysteresis.html"
        width="100%" height="500px" frameborder="0">
</iframe>
```

---

## Math Equations (MathJax / LaTeX)

**Inline math** — wrap with `\(` and `\)`:

The neutral axis depth is \( c = \frac{A_s f_y}{0.85 f'_c b} \).

**Block (display) math** — wrap with `$$`:

Nominal moment capacity of a rectangular section:

$$
M_n = A_s f_y \left( d - \frac{a}{2} \right)
$$

where \( a = \frac{A_s f_y}{0.85 f'_c b} \) is the equivalent stress block depth.

Equation of motion for a SDOF system:

$$
m\ddot{u} + c\dot{u} + ku = -m\ddot{u}_g(t)
$$

Damped natural frequency:

$$
\omega_d = \omega_n \sqrt{1 - \zeta^2}
$$

---

## Tables

| Parameter        | Symbol      | Value   | Units |
|------------------|-------------|---------|-------|
| Column height    | \( L \)     | 144     | in    |
| Axial load ratio | \( P/f'_cA_g \) | 0.10 | —   |
| Yield strength   | \( f_y \)   | 60      | ksi   |
| Concrete strength | \( f'_c \) | 5       | ksi   |

---

## Combining Elements

A typical workflow: run your analysis script, save the figure, then embed it.

```python
# 1. Run analysis and save figure
fig, ax = plt.subplots()
ax.plot(drift_history, force_history, 'k-', lw=0.8)
ax.set_xlabel('Drift Ratio (%)')
ax.set_ylabel('Base Shear (kip)')
fig.savefig('images/column-response.png', dpi=150, bbox_inches='tight')
```

Then in the post body:

```markdown
![Column response](/images/column-response.png)
*Figure 3: Lateral force–displacement response from OpenSees analysis.*
```
