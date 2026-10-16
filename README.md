# Do sinal ao espectro: um analisador e filtro de sinais

Sistematização de **Métodos Matemáticos** (CEUB): Séries de Fourier → Transformada de Fourier → Transformada de Laplace.

**Integrante(s):** Julio Cesar Sousa Oliveira (RA 22308014)

**Sinal escolhido:** um ECG real, o registro 208 do *MIT-BIH Arrhythmia Database* (PhysioNet), amostrado a 360 Hz. Somamos a ele uma interferência da rede elétrica de 60 Hz e 120 Hz.

**Problema:** tirar do sinal a deriva da linha de base, o ruído de 60 Hz e o ruído de alta frequência, sem deformar o complexo QRS.

## Estrutura do repositório

```
├── sistematizacao_sinais.ipynb   # notebook executável com as 4 etapas + relatório em células de texto
├── dados/
│   ├── ecg_mitbih208_360hz.csv   # sinal original (5 min, colunas tempo_s, ecg_mV)
│   └── ecg_filtrado.csv          # janela de 60 s: com interferência × filtrado
├── figuras/                      # todos os gráficos gerados pelo notebook (PNG)
├── requirements.txt
└── README.md
```

## Como executar

**Google Colab:** abra o `.ipynb` com *Arquivo → Fazer upload de notebook* (ou *GitHub → colar a URL do repositório*) e depois *Ambiente de execução → Executar tudo*. Se a pasta `dados/` não estiver lá, o notebook baixa o sinal sozinho.

**Local:**

```bash
pip install -r requirements.txt
```

```bash
jupyter notebook sistematizacao_sinais.ipynb
```

## O que cada etapa faz

| Etapa | Conteúdo | Principais resultados |
|---|---|---|
| **1. Série de Fourier** (Un. 1 e 2) | Onda quadrada: coeficientes calculados de forma simbólica (sympy) e numérica, somas parciais, forma complexa $c_n$, fenômeno de Gibbs, matriz de Gram (ortogonalidade) | $b_n = 4/(n\pi)$ para n ímpar; o overshoot converge para **8,95 %** do salto e não diminui com K; o erro RMS cai como $K^{-1/2}$ |
| **2. Espectro** (Un. 3) | FFT do ECG, eixo de frequência obtido de $f_s$, Nyquist, Δf, demonstração de *aliasing* | Frequência cardíaca de **≈ 98 bpm** no espectro e **99,5 bpm** contando os picos R no tempo; linhas em 60 e 120 Hz; reamostrando a 90 Hz sem anti-aliasing, as duas linhas se dobram para **30 Hz** |
| **3. Filtragem** (Un. 3) | Passa-altas 0,5 Hz + passa-baixas 40 Hz (Butterworth) + notch 60 Hz, aplicados como $Y = \lvert H\rvert X$ e reconstruídos pela **FFT inversa** | Rede de 60 Hz: **−56 dB**; 120 Hz: **−38 dB**; faixa útil: **−0,1 dB**; correlação do QRS **0,9999**; conferido com um filtro IIR digital independente (correlação 0,998) |
| **4. Laplace** (Un. 4) | RC de 1ª ordem resolvido no sympy ($h(t)$ e resposta ao degrau), $H(s)$ do filtro completo, mapa de polos e zeros, Bode, resposta ao impulso e ao degrau | 8 polos, todos com Re(s) < 0, portanto **estável**; os zeros em ±j2π·60 explicam o vale em 60 Hz visto no espectro |

## Roteiro sugerido para o vídeo (3–5 min)

| Tempo | O que mostrar | Fala-chave |
|---|---|---|
| 0:00–0:20 | Título e o sinal escolhido | "Vamos limpar um ECG real contaminado pela rede elétrica usando Fourier e Laplace." |
| 0:20–1:10 | `e1_somas_parciais.png` e `e1_gibbs.png` | "Somando harmônicos ímpares a onda quadrada aparece. O pico de Gibbs para em 9 % e não some; o que diminui é o erro médio." |
| 1:10–2:00 | `e2_sinal_tempo.png` e depois `e2_espectro.png` | "No tempo o ruído está misturado com o sinal. Na frequência ele vira duas linhas, em 60 e 120 Hz, e a deriva fica abaixo de 0,5 Hz. O batimento aparece em 1,6 Hz, cerca de 98 bpm." |
| 2:00–2:20 | `e2_aliasing.png` | "Amostrando abaixo de Nyquist, os 60 Hz caem dentro do ECG, em 30 Hz, e não há mais como separá-los." |
| 2:20–3:20 | `e3_antes_depois.png`, com zoom de 3 s | **Antes e depois:** "Multiplicamos o espectro por H e voltamos ao tempo com a FFT inversa. O ruído de 60 Hz sumiu e o QRS ficou intacto." |
| 3:20–4:30 | `e4_polos_zeros.png` e `e4_bode_vs_espectro.png` | "Os zeros sobre o eixo jω em 60 Hz produzem o vale que vimos no espectro. Os polos do Butterworth formam um semicírculo de raio 2π·40. Todos os polos estão no semiplano esquerdo, então o filtro é estável." |
| 4:30–5:00 | Conclusão | "O espectro e o mapa de polos e zeros contam a mesma história em duas linguagens." |

## Ferramentas

Python (Google Colab), `numpy`, `matplotlib`, `scipy.fft`, `scipy.signal`, `sympy`. Dados: PhysioNet / MIT-BIH, distribuídos via `scipy.datasets`.

**Referências:** Moody G.B., Mark R.G. *The impact of the MIT-BIH Arrhythmia Database*. IEEE Eng. Med. Biol. 20(3):45–50, 2001. Goldberger A. et al. *PhysioBank, PhysioToolkit, and PhysioNet*. Circulation 101(23):e215–e220, 2000.
