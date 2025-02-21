import streamlit as st
import matplotlib.pyplot as plt
from matplotlib.patches import Rectangle
import pandas as pd
from py packing import newPacker

def otimizar_corte_multiplas_chapas(chapa_largura, chapa_altura, pecas, maquina):
    # Configurações de margem e espaçamento
    if maquina == "Rauter":
        margem_lateral = 10
        espacamento = 12
    elif maquina == "Seccionadora":
        margem_lateral = 10
        espacamento = 4
    else:
        raise ValueError("Máquina não suportada.")
    # Dimensões ajustadas da chapa
    largura_ajustada = chapa_largura - 2 * margem_lateral
    altura_ajustada = chapa_altura - 2 * margem_lateral
    # Lista de todas as peças (com espaçamento)
    all_rects = []
    for idx, (w, h, q) in enumerate(pecas):
        for _ in range(q):
            all_rects.append((w + espacamento, h + espacamento, idx))
    # Verificar se as peças cabem na chapa
    for w, h, rid in all_rects:
        if (w > largura_ajustada and w > altura_ajustada) or (h > largura_ajustada and h > altura_ajustada):
            raise ValueError(f"Peça {rid+1} não cabe na chapa mesmo com rotação.")
    # Configurar o empacotador
    packer = newPacker(rotation=True)
    for _ in range(len(all_rects)):  # Chapas suficientes para o pior caso
        packer.add_bin(largura_ajustada, altura_ajustada)
    # Adicionar todas as peças
    for w, h, rid in all_rects:
        packer.add_rect(w, h, rid=rid)
    packer.pack()
    # Processar resultado
    plano_corte = []
    for bin_idx, abin in enumerate(packer):
        for rect in abin:
            x = rect.x + margem_lateral
            y = rect.y + margem_lateral
            largura = rect.width - espacamento
            altura = rect.height - espacamento
            plano_corte.append((bin_idx, x, y, largura, altura))
    return plano_corte, len(packer)

def desenhar_plano(plano_corte, chapa_largura, chapa_altura):
    # Agrupar peças por chapa
    chapas = {}
    for entrada in plano_corte:
        num_chapa, x, y, w, h = entrada
        if num_chapa not in chapas:
            chapas[num_chapa] = []
        chapas[num_chapa].append((x, y, w, h))
    # Plotar cada chapa separadamente
    for num_chapa, pecas in chapas.items():
        fig, ax = plt.subplots(figsize=(10, 7))
        ax.set_title(f"Chapa {num_chapa + 1}")
        ax.set_xlim(0, chapa_largura)
        ax.set_ylim(0, chapa_altura)
        
        for i, (x, y, w, h) in enumerate(pecas):
            cor = plt.cm.tab20(i % 20)
            ret = plt.Rectangle((x, y), w, h, edgecolor='black', facecolor=cor, alpha=0.6)
            ax.add_patch(ret)
            ax.text(x + w/2, y + h/2, f"{w}x{h}", ha='center', va='center', fontsize=8)
        
        plt.gca().invert_yaxis()
        st.pyplot(fig)

def processar_resultados(plano_corte, espessura_chapa, maquina, num_chapas, chapa_largura, chapa_altura):
    # Cálculo de aproveitamento
    area_total = chapa_largura * chapa_altura * num_chapas
    area_utilizada = sum(w * h for _, _, _, w, h in plano_corte)
    aproveitamento = (area_utilizada / area_total) * 100
    st.success(f"**Aproveitamento:** {aproveitamento:.2f}%")

def relatorio_fitas_e_cortes(plano_corte, espessura_chapa):
    total_metros = 0
    for _, x, y, w, h in plano_corte:
        perimetro = 2 * (w + h)
        total_metros += perimetro / 1000  # Convertendo para metros
    st.subheader("📏 Relatório de Cortes")
    st.write(f"Espessura da chapa: {espessura_chapa}mm")
    st.write(f"Metragem linear total de corte: {total_metros:.2f}m")
    st.write(f"Custo estimado: R${total_metros * 2.5:.2f} (considerando R$2,50/m)")  # Exemplo de custo

def gerar_etiquetas(plano_corte):
    st.subheader("🏷️ Etiquetas de Identificação")
    for idx, (num_chapa, x, y, w, h) in enumerate(plano_corte, 1):
        st.code(
            f"Peça {idx} - Chapa {num_chapa+1}\n"
            f"Posição: ({x+1}mm, {y+1}mm)\n"
            f"Dimensões: {w}x{h}mm", 
            language='plaintext'
        )

def exportar_csv_pdf(plano_corte, espessura_chapa, maquina, num_chapas, chapa_largura, chapa_altura):
    # CSV
    df = pd.DataFrame(
        [(chapa+1, x, y, w, h) for chapa, x, y, w, h in plano_corte],
        columns=["Chapa", "X (mm)", "Y (mm)", "Largura (mm)", "Altura (mm)"]
    )
    csv = df.to_csv(index=False)
    st.download_button("📥 Baixar CSV", csv, "plano_corte.csv", "text/csv")
    # PDF
    pdf = FPDF()
    pdf.add_page()
    pdf.set_font('Arial', 'B', 16)
    pdf.cell(0, 10, f"Plano de Corte - {maquina}", ln=1)
    pdf.set_font('Arial', '', 12)
    pdf.cell(0, 10, f"Chapas utilizadas: {num_chapas}", ln=1)
    # ... (restante do código PDF)

def main():
    st.set_page_config(page_title="Otimizador de Corte", layout="wide")
    with st.sidebar:
        # Configurações...
        st.write("Configurações...")
    
    tab1, tab2, tab3 = st.tabs(["📐 Plano de Corte", "📊 Relatórios", "⚙️ Configurações"])
    
    with tab1:
        # Visualização do plano...
        st.write("Visualização do plano...")
    
    with tab2:
        # Relatórios...
        st.write("Relatórios...")

if __name__ == "__main__":
    main()
