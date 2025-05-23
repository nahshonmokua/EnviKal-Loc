<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
</head>
<body>
  <h1>📡 EnviKal-Loc: Sub-10m Indoor LoRaWAN Localization using an Environmental-Aware Path Loss and Adaptive RSSI Smoothing}</h1>

  <h2>📌 Overview</h2>
  <p>
    This repository presents <strong>EnviKal-Loc</strong>, a robust indoor localization framework that fuses
    <strong>environmentally-augmented path loss modeling</strong> with <strong>adaptive Kalman filtering</strong>
    for precise distance estimation in <strong>LoRaWAN</strong> networks.
    Using a dataset of over <strong>1.3 million</strong> real-world measurements across six months,
    we model RSSI attenuation with environmental features and mitigate transient noise for sub-10m accuracy.
  </p>

  <h2>✨ Key Contributions</h2>
  <ul>
    <li>📈 <strong>MWM-EP:</strong> Multi-wall model extended with <em>temperature, humidity, CO₂, PM2.5, pressure</em>, and <em>SNR</em>.</li>
    <li>🔧 <strong>Kalman Filter:</strong> Per-device self-tuning 1D Kalman filter for smoothing RSSI volatility.</li>
    <li>🏗️ <strong>6-month real-world dataset:</strong> 1,328,334 measurements collected in a multi-room academic office setting.</li>
    <li>🎯 <strong>Distance Estimation:</strong> Achieves <strong>5.81 m MAE</strong>, outperforming traditional and ML-based methods.</li>
  </ul>

  <h2>📂 Repository Structure</h2>
  <ul>
    <li><code>data/</code> – Cleaned RSSI and environmental data logs (to be provided later).</li>
    <li><code>notebooks/</code> – Jupyter notebooks for modeling, filtering, and analysis.</li>
    <li><code>kalman/</code> – Python implementation of the self-tuning Kalman filter.</li>
    <li><code>models/</code> – Path loss models (MWM, MWM-EP, MWM-EP-KF).</li>
    <li><code>results/</code> – Visualizations and metrics (RMSE, MAE, R²).</li>
  </ul>

  <h2>🧪 Methodology</h2>
  <ol>
    <li><strong>Data Collection:</strong> LoRaWAN nodes with BME280, SCD41, and SPS30 sensors deployed indoors (EN1–EN6).</li>
    <li><strong>RSSI Filtering:</strong> Adaptive Kalman filter using innovation-based dynamic noise tuning.</li>
    <li><strong>Path Loss Modeling:</strong> Extended MLR combining physical walls, signal quality, and environment features.</li>
    <li><strong>Distance Estimation:</strong> Model inversion with filtered RSSI to estimate range.</li>
  </ol>

  <h2>📈 Results Summary</h2>
  <ul>
    <li>📉 Kalman filter reduced RSSI std. dev. from <strong>9.95 dB → 5.28 dB</strong></li>
    <li>📊 MAE (distance): MWM = 17.98 m → MWM-EP = 10.56 m → <strong>MWM-EP-KF = 5.81 m</strong></li>
    <li>🧠 R² increased from <strong>0.69 → 0.90</strong> with environmental & filtering enhancements</li>
  </ul>

  <h2>🚀 Reproduce the Analysis</h2>
  <pre><code>git clone https://github.com/yourusername/EnviKal-Loc.git
cd EnviKal-Loc
pip install -r requirements.txt
python run_analysis.py</code></pre>

  <h2>📜 Citation</h2>
  <p>If you use this work, please cite the associated paper:</p>
  <pre><code>@inproceedings{obiri2025envikal,
  title={EnviKal-Loc: Sub-10m Indoor LoRaWAN Localization using an Environmental-Aware Path Loss and Adaptive RSSI Smoothing},
  author={Nahshon Mokua Obiri and Kristof Van Laerhoven},
  booktitle={TBA 2025},
  year={2025},
  organization={TBA}
}
  </code></pre>

  <h2>📬 Contact</h2>
  <p>
    For questions, feedback, or collaboration inquiries, reach out via email: 📧 <strong>nahshon.obiri@student.uni-siegen.de</strong>
  </p>
</body>
</html>