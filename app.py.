import streamlit as st
from datetime import datetime
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import make_pipeline
from fpdf import FPDF
from PIL import Image, ImageDraw, ImageFont
import io
import requests
import textwrap
import tempfile
import os
import html

st.set_page_config(
    page_title="CIPAT Inteligente - ATAKADÃO ATAKAREJO",
    page_icon="🟠",
    layout="wide"
)

# ==================== IDENTIDADE VISUAL ====================
LARANJA = "#FF6600"
NOME_EMPRESA = "ATAKADÃO ATAKAREJO"

st.markdown("""
<style>
.main { background-color: #F8F9FA; }
.card-container {
    background-color: #FFFFFF;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.05);
    margin-bottom: 20px;
    border-left: 5px solid #FF6600;
}
h1, h2, h3 { color: #1F2D3D; }
.stButton > button {
    background-color: #FF6600 !important;
    color: white !important;
    border-radius: 8px;
    font-weight: bold;
    border: none;
    transition: all 0.3s ease;
}
.stButton > button:hover { background-color: #E55C00 !important; }
</style>
""", unsafe_allow_html=True)


# ==================== LOGIN ====================
def verificar_login():
    if "autenticado" not in st.session_state:
        st.session_state.autenticado = False

    if not st.session_state.autenticado:
        st.markdown("<br><br>", unsafe_allow_html=True)
        col1, col2, col3 = st.columns([1, 1.2, 1])
        with col2:
            st.markdown("""
                <div style="background:white; padding:35px; border-radius:15px;
                box-shadow:0 8px 24px rgba(0,0,0,0.1); border-top:6px solid #FF6600;
                text-align:center;">
                    <h1 style="color:#FF6600; margin-bottom:5px;">🟠 ATAKADÃO ATAKAREJO</h1>
                    <h3 style="color:#1F2D3D; font-size:18px; margin-bottom:15px;">
                        CIPAT Inteligente
                    </h3>
                    <p style="color:#666; font-size:13px; margin-bottom:25px;">
                        Acesso Restrito • Gestão de Segurança
                    </p>
                </div>
            """, unsafe_allow_html=True)

            usuario = st.text_input("👤 Usuário Corporativo", placeholder="Ex: admin ou rh")
            senha = st.text_input("🔑 Senha de Acesso", type="password")
            if st.button("🔓 Entrar no Sistema", use_container_width=True):
                if usuario.strip().lower() in ["admin", "rh", "cipat"] and senha == "atacadao2026":
                    st.session_state.autenticado = True
                    st.rerun()
                else:
                    st.error("Credenciais inválidas. Confira o usuário e a senha.")
            st.caption("Acesso demonstrativo: admin / atacadao2026")
        return False
    return True


if not verificar_login():
    st.stop()


# ==================== IA DE CLASSIFICAÇÃO ====================
@st.cache_resource
def carregar_ia():
    dados_treino = [
        ("Falta de uso de capacete, luvas ou oculos de protecao", "EPI Inadequado"),
        ("Funcionario sem botas de seguranca manipulando paletes ou cargas pesadas", "EPI Inadequado"),
        ("Chao escorregadio com oleo, agua ou produto derramado no corredor", "Risco de Queda"),
        ("Trabalho em altura nas prateleiras ou estoque sem cinto paraquedista", "Risco de Queda"),
        ("Cabos eletricos desencapados proximos a camara fria ou area umida", "Risco Eletrico"),
        ("Extintor de incendio vencido, despressurizado ou obstruido", "Risco de Incendio"),
        ("Colaborador com duvidas sobre prevencao de ISTs e preservativos", "Campanha de Saude e ISTs"),
        ("Orientacao sobre exames preventivos de cancer de mama e prostata", "Campanha de Saude e ISTs"),
        ("Sintomas de estresse extremo, ansiedade ou esgotamento mental", "Saude Mental e Bem-Estar"),
        ("Supervisor humilhando funcionario publicamente com gritos e xingamentos", "Assedio Moral ou Violencia"),
        ("Avancos indesejados e toque fisico inapropriado no trabalho", "Assedio Sexual"),
        ("Comentarios preconceituosos sobre raca, cor ou orientacao sexual", "Preconceito e Discriminacao"),
    ]
    textos = [item[0] for item in dados_treino]
    categorias = [item[1] for item in dados_treino]
    modelo = make_pipeline(TfidfVectorizer(ngram_range=(1, 2)), MultinomialNB())
    modelo.fit(textos, categorias)
    return modelo


modelo_ia = carregar_ia()

base_especialistas = {
    "EPI Inadequado": {
        "nr": "NR-06 (Equipamento de Proteção Individual)",
        "seguranca": "Interromper a atividade quando houver risco e orientar sobre o uso correto dos EPIs adequados.",
        "medico": "Avaliar eventual lesão ou desconforto e encaminhar ao serviço de saúde quando necessário.",
        "juridico": "Manter registros de entrega, orientação e reposição de EPIs e observar os requisitos legais aplicáveis.",
        "acao_cipa": "Acompanhar a situação e solicitar reposição dos equipamentos necessários."
    },
    "Risco de Queda": {
        "nr": "NR-35 (quando aplicável) / NR-08",
        "seguranca": "Isolar e sinalizar a área, eliminar o risco e impedir acesso até a correção.",
        "medico": "Em caso de acidente, acionar atendimento de primeiros socorros e avaliação de saúde.",
        "juridico": "Verificar os procedimentos e requisitos de segurança aplicáveis à atividade.",
        "acao_cipa": "Registrar a condição insegura e acompanhar a correção do piso, acesso ou proteção."
    },
    "Risco Eletrico": {
        "nr": "NR-10 (Segurança em Eletricidade)",
        "seguranca": "Isolar o local e acionar manutenção habilitada. Não tocar em cabos ou instalações expostas.",
        "medico": "Acionar atendimento de urgência em caso de choque elétrico.",
        "juridico": "Verificar se intervenções são realizadas por profissionais autorizados e capacitados.",
        "acao_cipa": "Impedir acesso de pessoas não autorizadas e acompanhar a correção."
    },
    "Risco de Incendio": {
        "nr": "NR-23 (Proteção Contra Incêndios)",
        "seguranca": "Manter rotas e equipamentos desobstruídos e acionar a equipe responsável para inspeção.",
        "medico": "Em caso de exposição à fumaça ou queimadura, acionar atendimento de emergência.",
        "juridico": "Verificar os procedimentos de prevenção, inspeção e resposta a emergências.",
        "acao_cipa": "Registrar a irregularidade e acompanhar a regularização dos equipamentos."
    },
    "Campanha de Saude e ISTs": {
        "nr": "Ações de saúde preventiva / PCMSO",
        "seguranca": "Disponibilizar informação educativa e orientar sobre os canais de saúde da empresa.",
        "medico": "Direcionar dúvidas clínicas ao serviço de saúde, preservando o sigilo médico.",
        "juridico": "Resguardar privacidade, confidencialidade e tratamento adequado de dados pessoais.",
        "acao_cipa": "Apoiar campanhas educativas e ações de prevenção."
    },
    "Saude Mental e Bem-Estar": {
        "nr": "Ações de prevenção e gerenciamento de riscos ocupacionais",
        "seguranca": "Acolher sem julgamento e encaminhar pelos canais internos apropriados.",
        "medico": "Orientar a busca por profissional de saúde qualificado, preservando a confidencialidade.",
        "juridico": "Tratar informações pessoais com confidencialidade e observar os procedimentos internos.",
        "acao_cipa": "Promover ações de prevenção, respeito e melhoria do ambiente de trabalho."
    },
    "Assedio Moral ou Violencia": {
        "nr": "Lei 14.457/2022 e procedimentos internos aplicáveis",
        "seguranca": "Priorizar a proteção das pessoas envolvidas e comunicar os canais responsáveis.",
        "medico": "Oferecer orientação para acesso a apoio médico ou psicológico, se necessário.",
        "juridico": "Encaminhar para apuração imparcial e sigilosa, evitando exposição e retaliação.",
        "acao_cipa": "Divulgar canais de acolhimento e acompanhar ações preventivas sem conduzir investigação por conta própria."
    },
    "Assedio Sexual": {
        "nr": "Lei 14.457/2022 / legislação aplicável",
        "seguranca": "Priorizar a segurança da pessoa que relata o fato e orientar sobre os canais formais.",
        "medico": "Disponibilizar orientação para acesso a apoio médico e psicológico especializado.",
        "juridico": "Encaminhar para apuração sigilosa, imparcial e conforme os procedimentos legais e internos.",
        "acao_cipa": "Reforçar prevenção, confidencialidade e proteção contra retaliação."
    },
    "Preconceito e Discriminacao": {
        "nr": "Legislação antidiscriminatória e políticas internas",
        "seguranca": "Interromper condutas discriminatórias e proteger as pessoas envolvidas.",
        "medico": "Orientar acesso a suporte de saúde quando solicitado.",
        "juridico": "Encaminhar para análise formal, confidencial e imparcial.",
        "acao_cipa": "Promover respeito, inclusão e ações educativas."
    }
}


def carregar_fonte(tamanho, negrito=False):
    caminhos = (
        ["C:/Windows/Fonts/arialbd.ttf", "C:/Windows/Fonts/segoeuib.ttf"]
        if negrito else
        ["C:/Windows/Fonts/arial.ttf", "C:/Windows/Fonts/segoeui.ttf"]
    )
    for caminho in caminhos:
        if os.path.exists(caminho):
            return ImageFont.truetype(caminho, tamanho)
    return ImageFont.load_default()


def limpar_pdf(texto):
    return str(texto).encode("latin-1", "replace").decode("latin-1")


# ==================== CABEÇALHO PRINCIPAL ====================
st.markdown("""
<div style="padding:10px 0; border-bottom:3px solid #FF6600; margin-bottom:20px;">
    <h1 style="color:#FF6600; margin-bottom:0;">🟠 ATAKADÃO ATAKAREJO</h1>
    <h2 style="color:#1F2D3D; margin-top:4px;">CIPAT Inteligente</h2>
    <p style="color:#555; font-size:16px;">
        Gestão Integrada de Segurança, Medicina do Trabalho e Conformidade Legal
    </p>
</div>
""", unsafe_allow_html=True)


# ==================== BARRA LATERAL ====================
with st.sidebar:
    st.markdown("### 🔍 Pesquisa Rápida (Google / NRs)")
    termo_busca = st.text_input("Digite sua dúvida ou norma:", placeholder="Ex: NR 12, saúde, CLT...")
    if termo_busca:
        from urllib.parse import quote_plus
        url_google = "https://www.google.com/search?q=" + quote_plus(termo_busca)
        st.markdown(f"[👉 Abrir busca no Google]({url_google})")

    st.markdown("---")
    st.markdown("### ⚙️ Painel de Controle")
    if st.button("🚪 Encerrar Sessão (Logout)", use_container_width=True):
        st.session_state.autenticado = False
        st.rerun()


# ==================== ABAS ====================
aba1, aba2 = st.tabs([
    "📋 Registro de Ocorrências (3 Assinaturas)",
    "🎨 Gerador de Cartazes Profissionais"
])

with aba1:
    st.markdown("""
    <div style="background:#FFFFFF; padding:25px; border-radius:10px;
    box-shadow:0 2px 8px rgba(0,0,0,0.04); margin-bottom:20px;
    border-left:5px solid #FF6600;">
        <h3 style="margin-top:0; color:#FF6600;">📝 Novo Registro Oficial de Ocorrência</h3>
        <p style="color:#666; font-size:14px;">
        Preencha os campos para registrar acidentes, incidentes ou relatos.
        A classificação automática é uma sugestão de apoio e deve ser validada
        pelos responsáveis. O PDF possui campos para assinaturas de RH,
        Gerência e CIPAT.
        </p>
    </div>
    """, unsafe_allow_html=True)

    col1, col2 = st.columns(2)
    with col1:
        nome_colaborador = st.text_input("👤 Nome do Colaborador Envolvido:", placeholder="Digite o nome completo...")
        setor = st.selectbox(
            "🏢 Setor / Seção:",
            ["Depósito / Estoque", "Frente de Caixa", "Câmara Fria",
             "Reposição", "Administrativo", "Outros"]
        )
    with col2:
        tipo_ocorrencia = st.selectbox(
            "⚠️ Tipo de Ocorrência:",
            ["Acidente de Trabalho / EPI", "Risco Operacional",
             "Assédio Moral", "Assédio Sexual", "Questão de Saúde",
             "Preconceito e Discriminação"]
        )
        data_ocorrencia = st.date_input("📅 Data do Ocorrido:")

    relato_detalhado = st.text_area(
        "📋 Relato Detalhado dos Fatos:",
        placeholder="Descreva com clareza o que aconteceu ou a situação observada...",
        height=120
    )

    if st.button("🚀 Processar com IA e Gerar PDF Oficial", use_container_width=True):
        if not nome_colaborador.strip() or not relato_detalhado.strip():
            st.warning("Por favor, preencha o nome do colaborador e o relato detalhado.")
        else:
            risco_detectado = modelo_ia.predict([relato_detalhado])[0]
            dados_esp = base_especialistas[risco_detectado]
            data_hora_emissao = datetime.now().strftime("%d/%m/%Y às %H:%M")
            st.success(f"Ocorrência classificada como: {risco_detectado}")
            st.info("A classificação automática é apenas apoio. A avaliação final deve ser feita por profissional responsável.")

            def gerar_pdf_tres_assinaturas():
                pdf = FPDF(orientation="P", unit="mm", format="A4")
                pdf.add_page()
                pdf.set_auto_page_break(auto=True, margin=15)

                pdf.set_font("Helvetica", "B", 13)
                pdf.cell(0, 8, limpar_pdf("ATAKADÃO ATAKAREJO • CIPAT INTELIGENTE"), ln=True, align="C")
                pdf.set_font("Helvetica", "B", 10)
                pdf.cell(0, 6, limpar_pdf(f"REGISTRO DE OCORRÊNCIA: {tipo_ocorrencia.upper()}"), ln=True, align="C")
                pdf.set_font("Helvetica", "I", 8)
                pdf.cell(0, 5, limpar_pdf(f"Emitido em: {data_hora_emissao}"), ln=True, align="C")
                pdf.ln(4)

                pdf.set_font("Helvetica", "B", 10)
                pdf.cell(0, 6, limpar_pdf("1. DADOS DO COLABORADOR E SETOR:"), ln=True)
                pdf.set_font("Helvetica", "", 9)
                pdf.multi_cell(0, 5, limpar_pdf(f"Colaborador: {nome_colaborador} | Setor: {setor}"))
                pdf.multi_cell(0, 5, limpar_pdf(f"Data do Fato: {data_ocorrencia.strftime('%d/%m/%Y')} | Classificação sugerida: {risco_detectado}"))
                pdf.ln(3)

                pdf.set_font("Helvetica", "B", 10)
                pdf.cell(0, 6, limpar_pdf("2. RELATO DETALHADO:"), ln=True)
                pdf.set_font("Helvetica", "", 9)
                pdf.multi_cell(0, 5, limpar_pdf(relato_detalhado))
                pdf.ln(3)

                pdf.set_font("Helvetica", "B", 10)
                pdf.multi_cell(0, 6, limpar_pdf(f"3. ORIENTAÇÕES DE REFERÊNCIA ({dados_esp['nr']}):"))
                for titulo, chave in [
                    ("Segurança do Trabalho", "seguranca"),
                    ("Medicina do Trabalho / SESMT", "medico"),
                    ("Apoio Jurídico / RH", "juridico"),
                    ("Acompanhamento CIPAT", "acao_cipa")
                ]:
                    pdf.set_font("Helvetica", "B", 9)
                    pdf.cell(0, 5, limpar_pdf(titulo + ":"), ln=True)
                    pdf.set_font("Helvetica", "", 9)
                    pdf.multi_cell(0, 4, limpar_pdf(dados_esp[chave]))
                pdf.ln(8)

                pdf.set_font("Helvetica", "B", 9)
                pdf.cell(0, 6, limpar_pdf("4. APROVAÇÃO E ASSINATURAS:"), ln=True)
                pdf.ln(14)
                y = pdf.get_y()
                assinaturas = [
                    (15, "Recursos Humanos (RH)"),
                    (80, "Gerência da Unidade"),
                    (145, "Representação CIPAT")
                ]
                for x, rotulo in assinaturas:
                    pdf.line(x, y, x + 50, y)
                    pdf.set_xy(x, y + 2)
                    pdf.set_font("Helvetica", "", 8)
                    pdf.cell(50, 4, limpar_pdf(rotulo), align="C")
                return bytes(pdf.output())

            pdf_bytes = gerar_pdf_tres_assinaturas()
            st.download_button(
                label="📥 Baixar PDF Oficial da Ocorrência (3 Assinaturas)",
                data=pdf_bytes,
                file_name=f"Registro_Ocorrencia_{datetime.now().strftime('%d%m%Y_%H%M')}.pdf",
                mime="application/pdf",
                use_container_width=True
            )


# ==================== GERADOR DE CARTAZES ====================
with aba2:
    st.markdown("""
    <div style="background:#FFFFFF; padding:25px; border-radius:10px;
    box-shadow:0 2px 8px rgba(0,0,0,0.04); margin-bottom:20px;
    border-left:5px solid #FF6600;">
        <h3 style="margin-top:0; color:#FF6600;">🎨 Gerenciador de Campanhas Visuais</h3>
        <p style="color:#666; font-size:14px;">
        Selecione um tema para gerar cartaz institucional, com opção de JPG e PDF A4.
        </p>
    </div>
    """, unsafe_allow_html=True)

    fotos_campanha = {
        "Uso Obrigatório de EPIs (Segurança Operacional)": {
            "url": "https://images.unsplash.com/photo-1504307651254-35680f356dfd?w=900",
            "cor": (255, 102, 0),
            "titulo": "SEGURANÇA EM PRIMEIRO LUGAR: USE SEUS EPIs",
            "texto": "Sua vida e sua integridade são prioridades. Utilize os equipamentos de proteção adequados à atividade e siga as orientações de segurança."
        },
        "Prevenção de ISTs e Saúde Integral": {
            "url": "https://images.unsplash.com/photo-1576091160399-112ba8d25d1d?w=900",
            "cor": (255, 102, 0),
            "titulo": "SAÚDE INTEGRAL E PREVENÇÃO DE ISTs",
            "texto": "Informação, respeito e prevenção são fundamentais. Procure os serviços de saúde e os canais de orientação disponíveis."
        },
        "Campanha de Saúde Preventiva (Câncer e Exames)": {
            "url": "https://images.unsplash.com/photo-1584515979956-d9f6e5d09982?w=900",
            "cor": (255, 102, 0),
            "titulo": "SAÚDE PREVENTIVA: CUIDE-SE",
            "texto": "Mantenha seus cuidados de saúde em dia e procure orientação profissional sobre exames preventivos adequados para você."
        },
        "Combate ao Assédio e Preconceito (Lei 14.457/2022)": {
            "url": "https://images.unsplash.com/photo-1521737604893-d14cc237f11d?w=900",
            "cor": (255, 102, 0),
            "titulo": "RESPEITO NO AMBIENTE DE TRABALHO",
            "texto": "Respeito, dignidade e empatia são inegociáveis. Não tolere assédio ou discriminação. Procure os canais formais de acolhimento."
        }
    }

    tipo_campanha = st.selectbox("🎯 Escolha o Tema da Campanha:", list(fotos_campanha.keys()))
    info_atual = fotos_campanha[tipo_campanha]

    try:
        resposta = requests.get(info_atual["url"], timeout=10)
        resposta.raise_for_status()
        foto_original = Image.open(io.BytesIO(resposta.content)).convert("RGB")
    except Exception:
        foto_original = Image.new("RGB", (900, 480), color=(230, 230, 230))
        aviso_imagem = True
    else:
        aviso_imagem = False

    foto_redimensionada = foto_original.resize((900, 480))
    cartaz = Image.new("RGB", (900, 900), color=(255, 255, 255))
    draw = ImageDraw.Draw(cartaz)

    draw.rectangle([(0, 0), (900, 75)], fill=(30, 30, 30))
    draw.rectangle([(0, 75), (900, 90)], fill=info_atual["cor"])
    draw.text((35, 24), "CIPAT INTELIGENTE • ATAKADÃO ATAKAREJO",
              fill=(255, 255, 255), font=carregar_fonte(22, True))
    cartaz.paste(foto_redimensionada, (0, 90))

    draw.rectangle([(0, 570), (900, 635)], fill=info_atual["cor"])
    titulo_font = carregar_fonte(22, True)
    draw.text((30, 590), info_atual["titulo"], fill=(255, 255, 255), font=titulo_font)

    linhas = textwrap.wrap(info_atual["texto"], width=68)
    pos_y = 665
    for linha in linhas:
        draw.text((35, pos_y), linha, fill=(40, 40, 40), font=carregar_fonte(20))
        pos_y += 32

    draw.rectangle([(0, 855), (900, 900)], fill=(240, 243, 246))
    draw.text((35, 870), "Material de Conscientização • CIPAT / RH / SESMT",
              fill=(90, 90, 90), font=carregar_fonte(16))

    buffer_jpg = io.BytesIO()
    cartaz.save(buffer_jpg, format="JPEG", quality=95)
    bytes_jpg = buffer_jpg.getvalue()

    def gerar_pdf_cartaz(imagem_cartaz):
        caminho_tmp = None
        try:
            with tempfile.NamedTemporaryFile(delete=False, suffix=".jpg") as tmp:
                imagem_cartaz.save(tmp.name, format="JPEG", quality=95)
                caminho_tmp = tmp.name
            pdf = FPDF(orientation="P", unit="mm", format="A4")
            pdf.add_page()
            pdf.image(caminho_tmp, x=10, y=12, w=190)
            return bytes(pdf.output())
        finally:
            if caminho_tmp and os.path.exists(caminho_tmp):
                os.remove(caminho_tmp)

    bytes_pdf = gerar_pdf_cartaz(cartaz)
    st.markdown("---")
    if aviso_imagem:
        st.warning("Não foi possível carregar a fotografia da internet. O cartaz foi gerado com fundo neutro.")
    st.image(cartaz, caption="Pré-visualização do Cartaz Institucional", use_container_width=True)

    col_jpg, col_pdf = st.columns(2)
    with col_jpg:
        st.download_button(
            label="🖼️ Baixar Cartaz em JPG (WhatsApp)",
            data=bytes_jpg,
            file_name=f"Cartaz_Atakadao_{datetime.now().strftime('%d%m%Y_%H%M')}.jpg",
            mime="image/jpeg",
            use_container_width=True
        )
    with col_pdf:
        st.download_button(
            label="🖨️ Baixar Cartaz em PDF (Impressão A4)",
            data=bytes_pdf,
            file_name=f"Cartaz_Atakadao_{datetime.now().strftime('%d%m%Y_%H%M')}.pdf",
            mime="application/pdf",
            use_container_width=True
        )

st.markdown("---")
st.markdown(
    "<p style='text-align:center; color:#888; font-size:13px;'>"
    "CIPAT Inteligente • ATAKADÃO ATAKAREJO • Segurança, Saúde e Prevenção"
    "</p>",
    unsafe_allow_html=True
)
