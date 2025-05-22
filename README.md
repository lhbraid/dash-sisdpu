# Relatório SISDPU - Unidade Fortaleza/CE

Dash-app para visualização interativa de PAJs da DPU em Fortaleza/CE.

## Estrutura do repositório

meu-dash-sisdpu/
├── app.py
├── requirements.txt
├── README.md
├── data/
│ └── planilhaSISDPU.xlsx
├── assets/
│ └── logo_dpu.png


Código original:

import pandas as pd
import dash
from dash import dcc, html, Input, Output
from dash import dash_table
import plotly.express as px
import os

# 1) Leitura do arquivo Excel
BASEDIR = os.path.dirname(__file__)
file_path = os.path.join(BASEDIR, "data", "planilhaSISDPU.xlsx")
df = pd.read_excel(file_path)

# 2) Filtrar apenas Fortaleza/CE
df["Unidade"] = df["Unidade"].astype(str).str.strip()
df = df[df["Unidade"].str.upper() == "FORTALEZA/CE"].copy()

# 3) Dicionários de mapeamento 
oficio = {
    "C. 01º OFÍCIO CRIMINAL": "01º OF CRIM",
    "C. 03º OFÍCIO CRIMINAL": "03º OF CRIM",
    "B. 04º OFÍCIO CÍVEL": "04º OF CÍVEL",
    "B. 05º OFÍCIO CÍVEL": "05º OF CÍVEL",
    "E. 02º OFÍCIO PREVIDENCIARIO": "02º OF PREV",
    "E. 01º OFÍCIO PREVIDENCIÁRIO": "01º OF PREV",
    "E. 04º OFÍCIO PREVIDENCIÁRIO": "04º OF PREV",
    "E. 03º OFÍCIO PREVIDENCIÁRIO": "03º OF PREV",
    "B. 06° OFÍCIO CÍVEL": "06° OF CÍVEL",
    "B. 01º OFÍCIO CÍVEL": "01º OF CÍVEL",
    "B. 03º OFÍCIO CÍVEL": "03º OF CÍVEL",
    "H. 1ª DEFENSORIA REGIONAL DE DIREITOS HUMANOS": "DRDH",
    "C. 02º OFÍCIO CRIMINAL": "02º OF CRIM",
    "B. 02º OFÍCIO CÍVEL": "02º OF CÍVEL",
    "R. 01º OFÍCIO REGIONAL": "01º OF REG",
    "R. 02º OFÍCIO REGIONAL": "02º OF REG",
    "R. 03º OFÍCIO REGIONAL": "03º OF REG"
}
pretensao = {
    "Criminal >> ELEITORAL (CRIMINAL)": "ELEITORAL (CRIMINAL)",
    "Criminal >> RESTITUIÇÃO DE BENS APREENDIDOS/SEQUESTRADOS": "RESTITUIÇÃO DE BENS APREENDIDOS/SEQUESTRADOS",
    "Cível >> CONSUMIDOR": "CONSUMIDOR",
    "Previdenciário >> Pensão por morte >> CONCESSÃO/RESTABELECIMENTO": "Pensão por morte",
    "Previdenciário >> AUXÍLIO-DOENÇA (BENEFÍCIO POR INCAPACIDADE TEMPORÁRIA) >> URBANA": "AUXÍLIO-DOENÇA",
    "Previdenciário >> BPC - BENEFÍCIO ASSISTENCIAL": "BPC",
    "Previdenciário >> RESTITUIÇÃO/COBRANÇA - BENEFÍCIO PREVIDENCIÁRIO RECEBIDO PELO(A) ASSISTIDO(A)": "RESTITUIÇÃO/COBRANÇA",
    "Cível >> Saúde >> MEDICAMENTOS E INSUMOS (RECLASSIFICAR)": "MEDICAMENTOS",
    "Cível >> FGTS / PIS >> LIBERAÇÃO DE VALORES": "FGTS / PIS",
    "Previdenciário >> Aposentadoria por tempo de contribuição >> REVISÃO": "Aposentadoria por tempo de contribuição",
    "Cível >> Tributário >> Execução Fiscal / Atendimento Inicial": "Tributário",
    "Cível >> Saúde >> TRATAMENTO FORA DE DOMICÍLIO (TFD)": "TFD",
    "Tutela Coletiva / Direitos Humanos >> Defesa Grupos Vulneráveis >> Vítimas de tortura": "Defesa Grupos Vulneráveis",
    "Criminal >> TRÁFICO DE DROGAS": "TRÁFICO DE DROGAS",
    "Previdenciário >> Aposentadoria por idade  >> REVISÃO": "Aposentadoria por idade",
    "Cível >> Tributário >> Impugnação Multas e Tributos": "Tributário",
    "Cível >> Saúde >> MEDICAMENTOS E INSUMOS REGISTRADOS NA ANVISA E NÃO PADRONIZADOS NO SUS": "MEDICAMENTOS",
    "Cível >> Saúde >> MEDICAMENTOS E INSUMOS REGISTRADOS NA ANVISA E PADRONIZADOS NO SUS": "MEDICAMENTOS",
    "Cível >> Saúde >> EXAMES, CONSULTAS E CIRURGIAS": "Exame, consultas e Cirurgia",
    "Cível >> EDUCAÇÃO >> TRANSFERÊNCIA E PROGRAMAS DE ASSISTÊNCIA ESTUDANTIL": "EDUCAÇÃO",
    "Criminal >> Crimes contra a dignidade sexual": "Crimes contra a dignidade sexual",
    "Previdenciário >> Aposentadoria por idade  >> Urbana": "Aposentadoria por idade",
    "Criminal >> Crimes contra o patrimônio >> Estelionato": "Crimes contra o patrimônio",
    "Criminal >> Crimes contra o Sistema Financeiro Nacional": "Crimes contra o Sistema Financeiro Nacional",
    "Criminal >> Crimes contra a administração pública  >> Peculato": "Crimes contra a administração pública",
    "Criminal >> Crimes contra a fé pública >> Falsidade documental": "Crimes contra a fé pública",
    "Cível >> COBRANÇA >> CONTRATOS BANCÁRIOS / ATENDIMENTO INICIAL": "COBRANÇA",
    "Criminal >> Execução Penal >> Penas restritivas de direito": "Execução Penal",
    "Cível >> Saúde >> TRATAMENTO ONCOLÓGICO": "Tratamento oncológico",
    "Cível >> EDUCAÇÃO >> PROGRAMAS DE BOLSAS E FINANCIAMENTO ESTUDANTIL (PROUNI, FIES)": "EDUCAÇÃO",
    "Cível >> CURADORIA ESPECIAL >> EXECUÇÃO FISCAL": "CURADORIA ESPECIAL",
    "Cível >> BENEFÍCIOS SOCIAIS >> BOLSA FAMÍLIA": "BENEFÍCIOS SOCIAIS",
    "Cível >> COBRANÇA >> DÍVIDA ATIVA NÃO TRIBUTÁRIA (INCLUSIVE EXECUÇÃO FISCAL)": "COBRANÇA",
    "Previdenciário >> Salário-maternidade >> Urbana": "Salário-maternidade",
    "Previdenciário >> Aposentadoria por idade  >> Rural": "Aposentadoria por idade",
    "Previdenciário >> APOSENTADORIA POR INVALIDEZ (APOSENTADORIA POR INCAPACIDADE PERMANENTE) >> Urbana": "APOSENTADORIA POR INVALIDEZ",
    "Cível >> Saúde >> INTERNAÇÕES E TRANSFERÊNCIAS HOSPITALARES": "Internação e Transferência",
    "Previdenciário >> APOSENTADORIA DA PESSOA COM DEFICIÊNCIA": "APOSENTADORIA DA PESSOA COM DEFICIÊNCIA",
    "Cível >> RESPONSABILIDADE CIVIL DO ESTADO": "RESPONSABILIDADE CIVIL DO ESTADO",
    "Criminal >> Execução Penal >> CUMPRIMENTO DE ACORDO DE NÃO PERSECUÇÃO PENAL": "Execução Penal",
    "Criminal >> ELEITORAL (CRIMINAL)": "ELEITORAL (CRIMINAL)",
    "Tutela Coletiva / Direitos Humanos >> Defesa Grupos Vulneráveis >> Comunidades tradicionais": "Defesa Grupos Vulneráveis",
    "Cível >> CONCURSO PÚBLICO >> PROVAS E EXAMES": "CONCURSO PÚBLICO",
    "Criminal >> Execução Penal >> Penitenciária Federal": "Execução Penal",
    "Criminal >> Crimes contra o patrimônio >> Roubo / Extorsão": "Crimes contra o patrimônio",
    "Cível >> MORADIA >> CONTRATOS HABITACIONAIS / SFH / MCMV / MCVA / PAR": "MORADIA",
    "Criminal >> Crimes contra a fé pública >> Moeda falsa": "Crimes contra a fé pública",
    "Cível >> SERVIDOR PÚBLICO CIVIL >> REMUNERAÇÃO E BENEFÍCIOS. REGIME ESTATUTÁRIO": "SERVIDOR PÚBLICO CIVIL",
    "Cível >> FGTS / PIS >> RESSARCIMENTO DE SAQUE INDEVIDO": "FGTS / PIS",
    "Cível >> EDUCAÇÃO >> CONCLUSÃO DE CURSO": "EDUCAÇÃO",
    "Cível >> ADMINISTRATIVO >> CONSELHOS PROFISSIONAIS": "ADMINISTRATIVO",
    "Previdenciário >> AUXÍLIO-DOENÇA (BENEFÍCIO POR INCAPACIDADE TEMPORÁRIA) >> REVISÃO": "AUXÍLIO-DOENÇA",
    "Previdenciário >> BPC - AUXÍLIO INCLUSÃO": "BPC - AUXÍLIO INCLUSÃO",
    "Cível >> RECONHECIMENTO DE CONDIÇÃO JURÍDICA INDIVIDUAL PARA EXERCÍCIO DE DIREITOS": "Reconhecimento de direito individual",
    "Previdenciário >> Aposentadoria por tempo de contribuição >> Urbana": "Aposentadoria por tempo de contribuição",
    "Cível >> ESTADUAL": "ESTADUAL",
    "Cível >> MORADIA >> DEFESA DA POSSE E DA PROPRIEDADE": "MORADIA",
    "Cível >> SEGURO DPVAT": "SEGURO DPVAT",
    "Tutela Coletiva / Direitos Humanos >> Defesa Grupos Vulneráveis >> Comunidades indígenas": "Defesa Grupos Vulneráveis",
    "Cível >> Internacional >> CARTA ROGATÓRIA": "Internacional",
    "Cível >> Saúde >> MEDICAMENTOS E INSUMOS SEM REGISTRO NA ANVISA": "MEDICAMENTOS",
    "Trabalhista >> Reconhecimento vínculo na CTPS": "Reconhecimento vínculo na CTPS",
    "Previdenciário >> BENEFÍCIOS SOCIAIS >> SEGURO DEFESO - PESCADOR ARTESANAL": "BENEFÍCIOS SOCIAIS",
    "Cível >> MORADIA >> ACESSO À MORADIA ADEQUADA": "MORADIA",
    "Previdenciário >> Aposentadoria por tempo de contribuição >> Rural": "Aposentadoria por tempo de contribuição",
    "Cível >> CONCURSO PÚBLICO >> AÇÕES AFIRMATIVAS": "CONCURSO PÚBLICO",
    "Cível >> Internacional >> ESTRANGEIRO": "Internacional",
    "Previdenciário >> APOSENTADORIA POR INVALIDEZ (APOSENTADORIA POR INCAPACIDADE PERMANENTE) >> REVISÃO": "APOSENTADORIA POR INVALIDEZ",
    "Cível >> Ambiental": "Ambiental",
    "Cível >> BENEFÍCIOS SOCIAIS >> AUXÍLIO EMERGENCIAL - COVID-19": "BENEFÍCIOS SOCIAIS",
    "Criminal >> Crimes Militares >> PRATICADO POR MILITAR": "Crimes Militares",
    "Criminal >> Crimes Militares >> PRATICADO POR CIVIL": "Crimes Militares",
    "Cível >> CURADORIA ESPECIAL >> CONTRATOS BANCÁRIOS": "CURADORIA ESPECIAL",
    "Previdenciário >> Auxílio-acidente": "Auxílio-acidente",
    "Criminal >> Crimes contra o patrimônio >> Furto": "Crimes contra o patrimônio",
    "Cível >> MILITAR >> REFORMA E AGREGAÇÃO": "MILITAR",
    "Criminal >> Crimes contra Ordem Tributária": "Crimes contra Ordem Tributária",
    "Cível >> Agrário": "Agrário",
    "Tutela Coletiva / Direitos Humanos >> Defesa Grupos Vulneráveis >> Trabalho escravo": "Defesa Grupos Vulneráveis",
    "Cível >> CONCURSO PÚBLICO >> CLASSIFICAÇÃO E PRETERIÇÃO": "CONCURSO PÚBLICO",
    "Criminal >> CRIMES CONTRA O MEIO-AMBIENTE": "CRIMES CONTRA O MEIO-AMBIENTE",
    "Cível >> EDUCAÇÃO >> PROCESSO SELETIVO (ENEM, SISU, INSTITUIÇÃO DE ENSINO FEDERAIS)": "EDUCAÇÃO",
    "Cível >> Internacional >> RETIRADA COMPULSÓRIA (EXPULSÃO, DEPORTAÇÃO E REPATRIAÇÃO)": "Internacional",
    "Cível >> CONCURSO PÚBLICO >> INSCRIÇÃO, REQUISITOS E DOCUMENTAÇÃO": "CONCURSO PÚBLICO",
    "Criminal >> Crimes contra a administração pública  >> Inserção de dados falsos (Art. 313-A e B)": "Crimes contra a administração pública",
    "Previdenciário >> Pensão por morte >> REVISÃO": "Pensão por morte",
    "Cível >> EDUCAÇÃO >> MATRÍCULA": "EDUCAÇÃO",
    "Cível >> BENEFÍCIOS SOCIAIS >> PASSE LIVRE": "BENEFÍCIOS SOCIAIS",
    "Criminal >> Crimes contra a administração pública  >> CONTRABANDO/DESCAMINHO": "Crimes contra a administração pública",
    "Cível >> ELEITORAL (CÍVEL)": "ELEITORAL (CÍVEL)",
    "Previdenciário >> Auxílio-reclusão": "Auxílio-reclusão",
    "Previdenciário >> AUXÍLIO-DOENÇA (BENEFÍCIO POR INCAPACIDADE TEMPORÁRIA) >> RURAL": "AUXÍLIO-DOENÇA",
    "Cível >> FGTS / PIS >> CORREÇÃO": "FGTS / PIS",
    "Criminal >> Execução Penal >> Privativas de liberdade - Penitenciária Estadual": "Execução Penal",
    "Criminal >> LAVAGEM DE DINHEIRO": "LAVAGEM DE DINHEIRO",
    "Cível >> MILITAR >> INGRESSO E PROCESSO SELETIVO": "MILITAR",
    "Criminal >> ASSISTÊNCIA INTERNACIONAL CRIMINAL": "ASSISTÊNCIA INTERNACIONAL CRIMINAL",
    "Cível >> CURADORIA ESPECIAL >> OUTROS": "CURADORIA ESPECIAL",
    "Criminal >> Crimes contra a organização do trabalho": "Crimes contra a organização do trabalho",
    "Cível >> EDUCAÇÃO >> AÇÕES AFIRMATIVAS (EDUCAÇÃO)": "EDUCAÇÃO",
    "Cível >> BENEFÍCIOS SOCIAIS >> AUXÍLIO BRASIL": "BENEFÍCIOS SOCIAIS",
    "Tutela Coletiva / Direitos Humanos >> Defesa Grupos Vulneráveis >> Migrações e refúgio": "Defesa Grupos Vulneráveis",
    "Cível >> COBRANÇA >> INDENIZAÇÃO AO ERÁRIO": "COBRANÇA",
    "Trabalhista >> Trabalhador urbano: outros": "Trabalhador urbano: outros",
    "Cível >> Tributário >> REPETIÇÃO DE INDÉBITO": "Tributário",
    "Cível >> BENEFÍCIOS SOCIAIS >> Seguro Desemprego": "BENEFÍCIOS SOCIAIS",
    "Criminal >> Crimes contra o patrimônio >> Receptação": "Crimes contra o patrimônio",
    "Cível >> SERVIDOR PÚBLICO CIVIL >> PAD / SINDICÂNCIA": "SERVIDOR PÚBLICO CIVIL",
    "Cível >> Internacional >> Subtração internacional de crianças": "Internacional",
    "Criminal >> Crimes contra a pessoa  >> AMEAÇA/CONSTRANGIMENTO ILEGAL": "Crimes contra a pessoa",
    "Previdenciário >> APOSENTADORIA POR INVALIDEZ (APOSENTADORIA POR INCAPACIDADE PERMANENTE) >> Rural": "APOSENTADORIA POR INVALIDEZ",
    "Cível >> Internacional >> ALIMENTOS INTERNACIONAIS": "Internacional",
    "Previdenciário >> Salário-maternidade >> Rural": "Salário-maternidade",
    "Criminal >> RÁDIO CLANDESTINA/TELECOMUNICAÇÕES": "RÁDIO CLANDESTINA/TELECOMUNICAÇÕES",
    "Cível >> MILITAR >> SERVIÇO MILITAR INICIAL": "MILITAR",
    "Previdenciário >> BENEFÍCIOS SOCIAIS >> SALÁRIO-FAMÍLIA": "BENEFÍCIOS SOCIAIS",
    "Previdenciário >> BENEFÍCIOS SOCIAIS >> SOLDADO DA BORRACHA (LEI 7.986/89)": "BENEFÍCIOS SOCIAIS",
    "Cível >> MILITAR >> ASSISTÊNCIA MÉDICO-HOSPITALAR": "MILITAR",
    "Tutela Coletiva / Direitos Humanos >> Saúde": "Saúde",
    "Tutela Coletiva / Direitos Humanos >> Moradia": "Moradia",
    "Criminal >> RACISMO": "RACISMO",
    "Criminal >> Crimes contra o patrimônio >> Dano": "Crimes contra o patrimônio",
    "Cível >> ADMINISTRATIVO >> IMPROBIDADE ADMINISTRATIVA/ATENDIMENTO INICIAL": "ADMINISTRATIVO",
    "Cível >> Internacional >> OPÇÃO DE NACIONALIDADE E NATURALIZAÇÃO": "Internacional",
    "Cível >> MILITAR >> LICENCIAMENTO. EXCLUSÃO. REINTEGRAÇÃO": "MILITAR"
}

materia = {
    "COMPETENCIA CRIMINAL": "Criminal",
    "COMPETÊNCIA CIVEL": "Civel",
    "COMPETENCIA PREVIDENCIARIA": "Previdenciaria",
    "DIREITOS HUMANOS-PRETENSÕES COLETIVAS": "DRDH",
    "TURMA RECURSAL - JEF": "Turma Recursal",
    "DPU PARA TODOS (35)": "Itinerante",
    "FAZENDA PÚBLICA ESTADUAL (ESTADO E MUNCÍPIO)": "Civel",
    "COMPETENCIA CÍVEL": "Civel"
}
colaborador = { 
    "Cristiano Alves de Sousa - Servidor CPRO": "CPRO",
    "Analista Criminal (Vanessa)": "Assessor(a)",
    "Cristiane de Paz Fernandes - Terceirizada DAT": "DAT",
    "Tiago Moreira dos Santos - Terceirizada DAT": "DAT",
    "Livia Mara França Lino Barros": "CPRO",
    "Ivens Moreira da Gama - Assessor Chefia": "Assessor(a)",
    "Larissa Lima Martiniano - Servidora CPRO": "CPRO",
    "Maria de Fatima Martins da Silva - Servidora CPRO": "CPRO",
    "Louise Nunes Novaes - Servidora CPRO": "CPRO",
    "Khrisna Luana Lino Nobre - Terceirizada DAT": "DAT",
    "Luiz Henrique Carvalho Braid - Servidor DAT": "DAT",
    "Gabriela Rebouças da Silva Paz - Terceirizada DAT": "DAT",
    "Lidya Maria de Gois Ary  - Servidora CPRO": "CPRO",
    "Tharrara Norens de Sousa Rodrigues (DRDH)": "DRDH",
    "Celine de Castro Coutinho (DRDH)": "DRDH",
    "Inacio Silva de Sousa (DSS)": "DSS",
    "Thiago Macedo Araujo (DRDH)": "DRDH",
    "Jose Fernandes da Silva Neto - Servidor CPRO": "CPRO",
    "Amanda Freitas Pontes Esteves - Servidora DAT": "DAT",
    "Oriel Rodrigues Filho - Servidor CADM": "CADM",
    "Jania Cristina Rolim Albuquerque - Servidora DAT": "DAT",
    "Bianca Caetano de Vasconcelos (DSS)": "DSS",
    "Melissa Mayumi Shirai Martins (DSS)": "DSS",
    "Analista Previdenciário (Moisés)": "Assessor(a)",
    "Mychelle Soares Lima Carvalho Caldas - Servidora CPRO": "CPRO",
    "Analista Previdenciário (Cecília)": "Assessor(a)",
    "Analista Cível (Márcia)": "Assessor(a)",
    "Lidia Ribeiro Nóbrega - DPF": "DPF",
    "Alex Feitosa de Oliveira - DPF": "DPF",
    "Filippe Augusto dos Santos Nascimento - DPF": "DPF",
    "Analista Cível  (Fatima  Feitosa)": "Assessor(a)",
    "Analista Cível (Carlos Augusto)": "Assessor(a)",
    "Analista Previdenciário (Isabel)": "Assessor(a)",
    "Analista Previdenciário (Moisés)": "Assessor(a)",
    "Analista Previdenciario (Flavia)": "Assessor(a)",
    "Gislene Frota Lima - DPF": "DPF",
    "Analista Cível (Kate)": "Assessor(a)",
    "Analista Cível (Márcia)": "Assessor(a)",
    "Analista Cível (Leonardo)": "Assessor(a)",
    "Edilson Santana Gonçalves Filho - DPF": "DPF",
    "Vanessa Pinheiro Nunes Martins - DPF": "DPF",
    "Daniel Kishita Albuquerque Bernardino - DPF": "DPF",
    "Carlos Eduardo Barbosa Paz - DPF": "DPF",
    "Erasmo Lopes Matias de Freitas - DPF": "DPF",
    "Marcelo Lopes Barroso - DPF": "DPF",
    "Lara Helen Melo das Neves": "Estagiario(a)",
    "Lidia Ribeiro Nóbrega - DPF": "DPF"
}

# --- 4) Transformações Gerais ---
# Ofício: remove tudo até o ponto, aplica mapeamento e title-case
df["Oficio"] = (
    df["Oficio"]
    .astype(str)
    .str.replace(r"^.*\.\s*", "", regex=True)
    .str.strip()
    .replace(oficio)
    .str.title()
    .replace("Drdh", "DRDH")
)

# Pretensão: strip, replace, capitalize
df["Pretensão"] = (
    df["Pretensão"]
    .astype(str)
    .str.strip()
    .replace(pretensao)
    .str.lower()
    .str.capitalize()
)

# Data de Abertura do PAJ: somente dd/mm/YYYY
df["Data de Abertura do PAJ"] = pd.to_datetime(
    df["Data de Abertura do PAJ"], dayfirst=True, errors="coerce"
).dt.strftime("%d/%m/%Y")

# Matéria: strip, replace, capitalize
df["Materia"] = (
    df["Materia"]
    .astype(str)
    .str.strip()
    .replace(materia)
    .str.lower()
    .str.capitalize()
    .replace("Drdh", "DRDH")
)

# Usuário: 1º e 2º nome (ou 3º se o 2º for “de”), title-case
def extract_usuario(full):
    parts = full.strip().split()
    if len(parts) >= 2:
        if parts[1].lower() == "de" and len(parts) >= 3:
            return " ".join(parts[:3])
        return " ".join(parts[:2])
    return parts[0] if parts else ""

df["Usuario"] = (
    df["Usuário que instaurou o paj"]
    .astype(str)
    .apply(extract_usuario)
    .str.title()
)

# Usuário2: adiciona código do colaborador (ou fallback)
def make_usuario2(full):
    parts = full.strip().split()
    if full in colaborador:
        if len(parts) >= 3 and parts[1].lower() == "de":
            base = " ".join(parts[:3])
        else:
            base = " ".join(parts[:2]) if len(parts) >= 2 else parts[0]
        return f"{base} - {colaborador[full].upper()}"
    if len(parts) > 2:
        if parts[1].lower() == "de":
            return " ".join(parts[:3])
        return " ".join([parts[0], parts[1], parts[-1]])
    return " ".join(parts) if parts else ""

df.insert(
    loc=df.columns.get_loc("Usuario") + 1,
    column="Usuario2",
    value=df["Usuário que instaurou o paj"].astype(str).apply(make_usuario2)
)

# Setor: extrai entre parênteses ou após "-", uppercase, corrige “OF…” e “A…”
s = df["Usuario2"].astype(str)
setor = s.str.extract(r"\((.*?)\)", expand=False)
sem = s.str.partition("-")[2].str.strip()
setor = setor.fillna(sem).str.upper()
setor = setor.where(~setor.str.startswith("OF"), "ESTAGIÁRIO(A)")
setor = setor.where(~setor.str.startswith("A"), "ASSESSOR(A)")
df.insert(
    loc=df.columns.get_loc("Usuario2") + 1,
    column="Setor",
    value=setor
)

# --- 5) Montagem do Dash ---
app = dash.Dash(__name__)
server = app.server

# Opções para filtros
oficio_opts  = [{"label": o, "value": o} for o in sorted(df["Oficio"].unique())]
pret_opts    = [{"label": p, "value": p} for p in sorted(df["Pretensão"].unique())]
materia_opts = [{"label": m, "value": m} for m in sorted(df["Materia"].unique())]
usuario_opts = [{"label": u, "value": u} for u in sorted(df["Usuario"].unique())]
# apenas setores oficiais
valid_setores = {v.upper() for v in colaborador.values()} | {"ESTAGIÁRIO(A)"}
setor_opts   = [
    {"label": s, "value": s}
    for s in sorted(df["Setor"].unique())
    if s in valid_setores
]

app.layout = html.Div([

    # --- HEADER FIXO COM LOGO, TÍTULO E CALENDÁRIO ---
    html.Div([
        html.Img(
            src=app.get_asset_url("logo_dpu.png"),
            style={"height":"60px"}
        ),
        html.Div([
    html.Div("Relatório SISDPU - Unidade Fortaleza/CE"),
    html.Div("Última atualização: 20/05/2025", style={"fontWeight": "normal", "fontSize": "16px"})
], style={
    "flex": 1,
    "textAlign": "center",
    "margin": 0,
    "fontWeight": "bold",
    "fontSize": "20px",
    "lineHeight": "1.2"
}),
        dcc.DatePickerRange(
            id="date-picker",
            start_date=pd.to_datetime(df["Data de Abertura do PAJ"], dayfirst=True).min(),
            end_date=pd.to_datetime(df["Data de Abertura do PAJ"], dayfirst=True).max(),
            display_format="DD/MM/YYYY",
            minimum_nights=0,             # permite start_date == end_date
            # allow_single_day_range=True,  # autoriza range de um único dia
            style={
                "fontSize":"12px",
                "height":"32px",
                "minWidth":"200px",
                "marginLeft":"auto"
            },
            calendar_orientation="horizontal"
        )
    ], style={
        "display":"flex",
        "alignItems":"center",
        "padding":"10px 20px",
        "backgroundColor":"#f5f5f5",
        "borderBottom":"1px solid #ddd",
        "position":"fixed",
        "top":0, "left":0, "width":"100%", "zIndex":1000
    }),

    # --- COLUNA DE FILTROS (SEM CALENDÁRIO) ---
    html.Div([

        html.Div([
            html.H4(
                "Qtd total de PAJs instaurados",
                style={"textAlign":"center","margin":0,"fontSize":"14px"}
            ),
            html.P(
                f"{len(df):,}",
                style={"textAlign":"center","fontSize":"16px","margin":"0 0 6px 0"}
            )
        ], style={
            "border":"1px solid #ccc",
            "borderRadius":"5px",
            "padding":"6px",
            "marginTop":"10px",
            "marginBottom":"12px",
            "backgroundColor":"#fafafa"
        }),

        html.H4("Filtros", style={"textAlign":"center","fontSize":"16px","margin":"0 0 8px 0"}),

        html.Label("Ofício",    style={"textAlign":"center","margin":"4px 0"}),
        dcc.Dropdown(id="oficio-filter",    options=oficio_opts,  multi=True, style={"fontSize":"12px","height":"32px"}),
        html.Br(),

        html.Label("Pretensão", style={"textAlign":"center","margin":"4px 0"}),
        dcc.Dropdown(id="pretensao-filter", options=pret_opts,   multi=True, style={"fontSize":"12px","height":"32px"}),
        html.Br(),

        html.Label("Matéria",   style={"textAlign":"center","margin":"4px 0"}),
        dcc.Dropdown(id="materia-filter",  options=materia_opts, multi=True, style={"fontSize":"12px","height":"32px"}),
        html.Br(),

        html.Label("Usuário",   style={"textAlign":"center","margin":"4px 0"}),
        dcc.Dropdown(id="usuario-filter",  options=usuario_opts, multi=True, style={"fontSize":"12px","height":"32px"}),
        html.Br(),

        html.Label("Setor",     style={"textAlign":"center","margin":"4px 0"}),
        dcc.Dropdown(id="setor-filter",    options=setor_opts,    multi=True, style={"fontSize":"12px","height":"32px"}),

    ], style={
        "position":"fixed",
        "top":"80px",
        "left":0,
        "width":"16%",
        "height":"calc(100vh - 80px)",
        "padding":"12px",
        "overflowY":"auto",
        "fontSize":"12px"
    }),

    # --- PAINEL DE GRÁFICOS E TABELAS ---
    html.Div([

        dcc.Graph(id="oficio-bar-chart"),

        html.H4("TOP 10 Pretensões", style={"textAlign":"center"}),
        dash_table.DataTable(
            id="pretensao-table",
            columns=[
                {"name":"Pretensão","id":"Pretensão"},
                {"name":"Quantidade","id":"Quantidade"},
                {"name":"Percentual (%)","id":"Percentual"}
            ],
            data=[],
            style_table={"overflowX":"auto"},
            style_header={"textAlign":"center"},
            style_cell_conditional=[
                {"if":{"column_id":"Quantidade"},"textAlign":"center"},
                {"if":{"column_id":"Percentual"},"textAlign":"center"}
            ],
            style_cell={"textAlign":"left"}
        ),

        dcc.Graph(id="materia-bar-chart"),

        html.H4("TOP 10 Usuários", style={"textAlign":"center"}),
        dash_table.DataTable(
            id="usuario-table",
            columns=[
                {"name":"Usuário","id":"Usuario"},
                {"name":"Quantidade","id":"Quantidade"},
                {"name":"Percentual (%)","id":"Percentual"}
            ],
            data=[],
            style_table={"overflowX":"auto"},
            style_header={"textAlign":"center"},
            style_cell_conditional=[
                {"if":{"column_id":"Quantidade"},"textAlign":"center"},
                {"if":{"column_id":"Percentual"},"textAlign":"center"}
            ],
            style_cell={"textAlign":"left"}
        ),

        html.H4("", style={"textAlign":"center"}),
        dcc.Graph(id="setor-donut"),

    ], style={
        "marginLeft":"18%",
        "marginTop":"80px",
        "width":"80%",
        "padding":"20px",
        "height":"calc(100vh - 80px)",
        "overflowY":"auto"
    })

])

@app.callback(
    Output("oficio-bar-chart",  "figure"),
    Output("pretensao-table",   "data"),
    Output("materia-bar-chart", "figure"),
    Output("usuario-table",     "data"),
    Output("setor-donut",       "figure"),
    Input("date-picker",        "start_date"),
    Input("date-picker",        "end_date"),
    Input("oficio-filter",      "value"),
    Input("pretensao-filter",   "value"),
    Input("materia-filter",     "value"),
    Input("usuario-filter",     "value"),
    Input("setor-filter",       "value"),
)
def update_dashboard(start_date, end_date, sel_of, sel_pret, sel_mat, sel_user, sel_setor):
    dff = df.copy()

    # filtrar datas
    if start_date:
        dff = dff[pd.to_datetime(dff["Data de Abertura do PAJ"], dayfirst=True) >= pd.to_datetime(start_date)]
    if end_date:
        dff = dff[pd.to_datetime(dff["Data de Abertura do PAJ"], dayfirst=True) <= pd.to_datetime(end_date)]

    # filtros dropdown
    if sel_of:     dff = dff[dff["Oficio"].isin(sel_of)]
    if sel_pret:   dff = dff[dff["Pretensão"].isin(sel_pret)]
    if sel_mat:    dff = dff[dff["Materia"].isin(sel_mat)]
    if sel_user:   dff = dff[dff["Usuario"].isin(sel_user)]
    if sel_setor:  dff = dff[dff["Setor"].isin(sel_setor)]
    dff = dff[~dff["Setor"].str.fullmatch(r"\d+")]

    # correção extra de DRDH
    dff["Oficio"] = dff["Oficio"].replace({
        "1ª Defensoria Regional De Direitos Humanos": "DRDH"
    })

    # gráfico Ofício
    oficio_counts = dff["Oficio"].value_counts()
    total_of = oficio_counts.sum()
    categorias = sorted(oficio_counts.index, key=lambda s: s.split()[-1].lower())
    fig_of = px.bar(
        x=oficio_counts.index,
        y=oficio_counts.values,
        color=oficio_counts.index,
        labels={"x":"Ofício","y":"Quantidade"},
        title="<b>Distribuição por Ofício</b>",
        text=[f"{v} ({v/total_of:.1%})" for v in oficio_counts.values],
        height=600      # <— define a altura do gráfico
    )
    fig_of.update_layout(
        title_x=0.5,
        xaxis={"categoryorder":"array","categoryarray":categorias},
        showlegend=True,
        legend_title_text="Ofício",
        legend_orientation="v",
        margin={"r":200}  # espaço extra para a legenda
    )
    fig_of.update_traces(textposition="outside")

    # tabela Pretensão
    pret_counts = dff["Pretensão"].value_counts()
    top10 = pret_counts.head(10)
    pret_df = pd.DataFrame({
        "Pretensão": top10.index,
        "Quantidade": top10.values,
        "Percentual": (top10.values / pret_counts.sum() * 100).round(1)
    })

    # gráfico Matéria
    mat_counts = dff["Materia"].value_counts().sort_index()
    total_mat = mat_counts.sum()
    fig_mat = px.bar(
        x=mat_counts.index,
        y=mat_counts.values,
        color=mat_counts.index,
        labels={"x":"Matéria","y":"Quantidade"},
        title="<b>Distribuição por Matéria</b>",
        text=[f"{v} ({v/total_mat:.1%})" for v in mat_counts.values]
    )
    fig_mat.update_layout(
        title_x=0.5,
        xaxis={"categoryorder":"array","categoryarray":sorted(mat_counts.index)},
        margin={"t":80, "b":40, "r":200}
    )
    fig_mat.update_traces(textposition="outside")

    # tabela Usuário
    user_counts = dff["Usuario"].value_counts()
    top10_user = user_counts.head(10)
    user_df = pd.DataFrame({
        "Usuario": top10_user.index,
        "Quantidade": top10_user.values,
        "Percentual": (top10_user.values / user_counts.sum() * 100).round(1)
    })

    # gráfico Setor (rosca)
    setor_counts = (
        dff["Setor"]
        .astype(str)
        .str.strip()
        .value_counts()
    )
    setor_counts = setor_counts[setor_counts.index.str.contains(r"[A-Za-zÀ-ÿ]")]
    # 2) remove rótulos puramente numéricos (ex.: "7")
    setor_counts = setor_counts.drop(labels=["7"], errors="ignore")

    # 3) total de itens atualmente filtrados em 'Setor'
    total_setor = int(setor_counts.sum())

    # 4) monta o gráfico de rosca
    fig_setor = px.pie(
        names=setor_counts.index,
        values=setor_counts.values,
        hole=0.4,
        title="<b>Distribuição por Setor</b>"
    )
    fig_setor.update_traces(textinfo="value+percent")

    # 5) adiciona anotação com o total à esquerda do donut
    fig_setor.update_layout(
        title_x=0.5,
        annotations=[
            dict(
                x=-0.2,               # à esquerda do centro
                y=0.5,                # central verticalmente
                text=f"<b>Total: {total_setor:,}</b>",
                showarrow=False,
                font=dict(size=14)
            )
        ]
    )
    return fig_of, pret_df.to_dict("records"), fig_mat, user_df.to_dict("records"), fig_setor

if __name__ == "__main__":
    app.run(debug=True, port=8055)
