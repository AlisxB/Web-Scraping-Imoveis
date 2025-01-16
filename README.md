# Documentação do Projeto de Web Scraping de Imóveis em Fortaleza

## Visão Geral
Este projeto realiza web scraping no site [Guimarães Imóveis](https://www.guimaraesimoveis.com.br/imoveis/a-venda/fortaleza) para extrair informações detalhadas sobre imóveis à venda em Fortaleza. Os dados extraídos são salvos em uma planilha Excel para análise posterior.

## Tecnologias Utilizadas
- Python
- Selenium: Para automatizar a navegação no site e interagir com elementos dinâmicos.
- BeautifulSoup: Para fazer a extração de dados do HTML das páginas.
- Pandas: Para estruturar e salvar os dados em uma planilha Excel.

## Estrutura do Projeto

### 1. Configuração Inicial
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup
import pandas as pd
from datetime import datetime
import time

driver = webdriver.Chrome()

# URL da página
url = "https://www.guimaraesimoveis.com.br/imoveis/a-venda/fortaleza"
driver.get(url)

# Lista para armazenar os dados extraídos
data = []

max_results = 100  # Limite para número de resultados
```

### 2. Função para Extrair Dados de uma Página
```python
def extrair_dados_pagina(soup):
    global data
    cards = soup.find_all('div', class_='card')
    for card in cards:
        if len(data) >= max_results:
            return
        try:
            # Hiperlink do imóvel
            link = card.find('a', href=True)
            link_text = f"https://www.guimaraesimoveis.com.br{link['href']}" if link else "N/A"

            # Bairro
            bairro = card.find('h2', class_='card-title')
            bairro_text = bairro.get_text(strip=True) if bairro else "N/A"

            # Descrição do imóvel
            titulo = card.find('p', class_='card-text')
            titulo_text = titulo.get_text(strip=True) if titulo else "N/A"

            # Preço do imóvel
            price = card.find('span', class_='h-money location')
            price_text = price.get_text(strip=True) if price else "N/A"

            # Valor do condomínio
            condominio = card.find('span', text=lambda t: 'Condomínio' in t if t else False)
            condominio_text = condominio.get_text(strip=True).replace('Condomínio', '').strip() if condominio else "N/A"

            # Valor do IPTU
            iptu = card.find('span', text=lambda t: 'IPTU' in t if t else False)
            iptu_text = iptu.get_text(strip=True).replace('IPTU', '').strip() if iptu else "N/A"

            # Extrair valores de Quartos, Suítes, Banheiros, Vagas, e Área
            values = card.find_all('div', class_='value')
            quartos_text = suites_text = banheiros_text = vagas_text = area_text = "N/A"

            # Descrição adicional
            hidden_description = card.find('p', class_='description hidden-sm-down')
            hidden_description_text = hidden_description.get_text(strip=True) if hidden_description else "N/A"

            for value in values:
                label = value.find('br').next_sibling.strip()
                val = value.find('span', class_='h-money').get_text(strip=True)
                if "quartos" in label:
                    quartos_text = val
                elif "suíte" in label:
                    suites_text = val
                elif "banheiros" in label:
                    banheiros_text = val
                elif "vaga" in label:
                    vagas_text = val
                elif "m²" in label:
                    area_text = val

            # Adicionar os dados extraídos à lista
            data.append({
                "Bairro": bairro_text,
                "Título": titulo_text,
                "Valor": price_text,
                "Condomínio": condominio_text,
                "IPTU": iptu_text,
                "Quartos": quartos_text,
                "Suítes": suites_text,
                "Banheiros": banheiros_text,
                "Vagas": vagas_text,
                "Área (m²)": area_text,
                "Descrição Adicional": hidden_description_text,
                "Link": link_text
            })
        except Exception as e:
            print(f"Erro ao extrair dados de um imóvel: {e}")
```

### 3. Função para Carregar Mais Resultados
```python
def carregar_mais_resultados():
    try:
        wait = WebDriverWait(driver, 10)
        button = wait.until(EC.element_to_be_clickable((By.CLASS_NAME, 'btn-next')))
        ActionChains(driver).move_to_element(button).click(button).perform()
        time.sleep(3)
    except Exception as e:
        print("Não foi possível carregar mais resultados:", e)
```

### 4. Loop para Navegar pelas Páginas e Extrair Dados
```python
while len(data) < max_results:
    soup = BeautifulSoup(driver.page_source, 'html.parser')
    extrair_dados_pagina(soup)
    
    if len(data) >= max_results:
        break
    try:
        carregar_mais_resultados()
    except:
        print("Não há mais resultados para carregar.")
        break

driver.quit()
```

### 5. Salvando os Dados em uma Planilha Excel
```python
# Criar um DataFrame com resultados
df = pd.DataFrame(data)

# Salvando dados em planilha Excel com a data e hora no nome do arquivo
current_time = datetime.now().strftime("%Y%m%d_%H%M%S")
file_name = f"imoveis_fortaleza_{current_time}.xlsx"
df.to_excel(file_name, index=False)

print(f"Dados salvos em '{file_name}'")
```

## Como Executar o Projeto
1. Certifique-se de ter o [Python](https://www.python.org/downloads/) instalado.
2. Instale as bibliotecas necessárias com o comando:
   ```bash
   pip install selenium beautifulsoup4 pandas
   ```
3. Baixe o [chromedriver](https://sites.google.com/chromium.org/driver/) compatível com sua versão do Google Chrome e adicione-o ao PATH do sistema.
4. Execute o script Python fornecido.

## Considerações
- O script é configurado para extrair até 100 resultados, mas esse valor pode ser ajustado.
- A estrutura do site pode mudar, exigindo ajustes no código.

## Licença
Este projeto é licenciado sob a [Licença MIT](LICENSE).
