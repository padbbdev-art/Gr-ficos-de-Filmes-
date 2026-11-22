import pandas as pd
import matplotlib.pyplot as plt
import os

os.makedirs("resultados", exist_ok=True)

filmes = pd.read_csv("movies.csv")
avaliacoes = pd.read_csv("ratings.csv")

filmes = filmes.dropna()
avaliacoes = avaliacoes.dropna()

dados = pd.merge(avaliacoes, filmes, on="movieId")

generos = dados.assign(
    genero=dados["genres"].str.split("|")
).explode("genero")

traducao_generos = {
    "Action": "Ação",
    "Adventure": "Aventura",
    "Animation": "Animação",
    "Children": "Infantil",
    "Comedy": "Comédia",
    "Crime": "Crime",
    "Documentary": "Documentário",
    "Drama": "Drama",
    "Fantasy": "Fantasia",
    "Film-Noir": "Filme noir",
    "Horror": "Terror",
    "IMAX": "IMAX",
    "Musical": "Musical",
    "Mystery": "Mistério",
    "Romance": "Romance",
    "Sci-Fi": "Ficção científica",
    "Thriller": "Suspense",
    "War": "Guerra",
    "Western": "Faroeste",
    "(no genres listed)": "Sem gênero informado"
}

generos["genero"] = generos["genero"].replace(traducao_generos)

filmes_analisados = dados.groupby(
    ["movieId", "title"]
).agg(
    media_nota=("rating", "mean"),
    quantidade_notas=("rating", "count")
).reset_index()

melhores = filmes_analisados[
    filmes_analisados["quantidade_notas"] >= 20
].sort_values("media_nota", ascending=False).head(10)

mais_avaliados = filmes_analisados.sort_values(
    "quantidade_notas", ascending=False
).head(10)

por_genero = generos.groupby("genero").agg(
    media_nota=("rating", "mean"),
    quantidade_filmes=("movieId", "nunique")
).reset_index()

notas = dados["rating"].value_counts().sort_index()

melhores.to_csv("resultados/melhores_filmes.csv", index=False)
mais_avaliados.to_csv("resultados/filmes_mais_avaliados.csv", index=False)
por_genero.to_csv("resultados/avaliacoes_por_genero.csv", index=False)

plt.figure(figsize=(10, 5))
plt.barh(melhores["title"], melhores["media_nota"])
plt.title("Filmes com maior nota média")
plt.xlabel("Nota média")
plt.xlim(0, 5)
plt.tight_layout()
plt.savefig("resultados/01_melhores_filmes.png")
plt.close()

plt.figure(figsize=(10, 5))
plt.barh(mais_avaliados["title"], mais_avaliados["quantidade_notas"])
plt.title("Filmes com mais avaliações")
plt.xlabel("Quantidade de avaliações")
plt.tight_layout()
plt.savefig("resultados/02_filmes_mais_avaliados.png")
plt.close()

tabela = por_genero.sort_values("media_nota")
plt.figure(figsize=(9, 6))
plt.barh(tabela["genero"], tabela["media_nota"])
plt.title("Nota média por gênero")
plt.xlabel("Nota média")
plt.ylabel("Gênero")
plt.xlim(0, 5)
plt.tight_layout()
plt.savefig("resultados/03_media_por_genero.png")
plt.close()

plt.figure(figsize=(7, 5))
plt.bar(notas.index.astype(str), notas.values)
plt.title("Distribuição das avaliações")
plt.xlabel("Nota")
plt.ylabel("Quantidade")
plt.tight_layout()
plt.savefig("resultados/04_distribuicao_notas.png")
plt.close()

quantidade_generos = generos.groupby("genero")["movieId"].nunique().sort_values()
plt.figure(figsize=(9, 6))
plt.barh(quantidade_generos.index, quantidade_generos.values)
plt.title("Quantidade de filmes por gênero")
plt.xlabel("Quantidade de filmes")
plt.ylabel("Gênero")
plt.tight_layout()
plt.savefig("resultados/05_filmes_por_genero.png")
plt.close()

print("Análise concluída!")
print("Filmes:", len(filmes))
print("Avaliações:", len(avaliacoes))
print("Nota média geral:", round(dados["rating"].mean(), 2))
print("Resultados salvos na pasta resultados.")
