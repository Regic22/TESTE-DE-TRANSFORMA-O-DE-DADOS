import pdfplumber
import pandas as pd
import zipfile
import os

def extrair_tabela(pdf_path):
    dados = []
    with pdfplumber.open(pdf_path) as pdf:
        for pagina in pdf.pages:
            tabelas = pagina.extract_tables()
            for tabela in tabelas:
                for linha in tabela:
                    dados.append(linha)
    return dados

def salvar_csv(dados, nome_csv):
    colunas = dados[0]  # Assume que a primeira linha contém os cabeçalhos
    df = pd.DataFrame(dados[1:], columns=colunas)
    
    # Substituir abreviações conforme legenda
    legenda = {"OD": "Odontológico", "AMB": "Ambulatorial"}
    df.replace(legenda, inplace=True)
    
    df.to_csv(nome_csv, index=False)
    print(f"CSV salvo como {nome_csv}")

def compactar_csv(nome_csv, nome_zip):
    with zipfile.ZipFile(nome_zip, 'w') as zipf:
        zipf.write(nome_csv)
        os.remove(nome_csv)  # Remove o CSV após compactação
    print(f"Arquivo compactado em {nome_zip}")

def main():
    pdf_anexo = "Anexo_I.pdf"  # Arquivo baixado no teste 1
    nome_csv = "Rol_de_Procedimentos.csv"
    nome_zip = "Teste_SeuNome.zip"  # Substitua SeuNome pelo seu nome
    
    dados = extrair_tabela(pdf_anexo)
    if not dados:
        print("Nenhuma tabela extraída!")
        return
    
    salvar_csv(dados, nome_csv)
    compactar_csv(nome_csv, nome_zip)
    
if __name__ == "__main__":
    main()
