# sistema_de_vendas
import os
import random
import string

USERS_FILE = 'usuarios.txt'
PRODUCTS_FILE = 'produtos.txt'

def gerar_senha(comprimento=8, incluir_maiusculo=True, incluir_minusculo=True,
                incluir_numeros=True, incluir_caracter=False):
    caracteres = ''
    if incluir_minusculo:
        caracteres += string.ascii_lowercase
    if incluir_maiusculo:
        caracteres += string.ascii_uppercase
    if incluir_numeros:
        caracteres += string.digits
    if incluir_caracter:
        caracteres += "!@#$%&*()-_=+[]{};:,.<>?/"
    if not caracteres:
        caracteres = string.ascii_letters + string.digits
    return ''.join(random.choice(caracteres) for _ in range(comprimento))

def ensure_files():
    for f in (USERS_FILE, PRODUCTS_FILE):
        if not os.path.exists(f):
            open(f, 'a', encoding='utf-8').close()

def cadastrar_usuario():
    email = input("Digite seu email: ").strip()
    escolha = input("Deseja gerar uma senha automaticamente? (s/n): ").strip().lower()
    if escolha == 's':
        senha = gerar_senha()
        print(f"Sua senha gerada: {senha}")
    else:
        senha = input("Digite sua senha: ").strip()
    with open(USERS_FILE, 'a', encoding='utf-8') as u:
        u.write(f'{email},{senha}\n')
    print("Usuário cadastrado com sucesso!")

def login():
    email = input("Digite seu email: ").strip()
    senha = input("Digite sua senha: ").strip()
    try:
        with open(USERS_FILE, 'r', encoding='utf-8') as u:
            for line in u:
                parts = line.strip().split(',', 1)
                if len(parts) != 2:
                    continue
                email_arquivo, senha_arquivo = parts
                if email == email_arquivo and senha == senha_arquivo:
                    print("Login realizado com sucesso!")
                    return True
    except FileNotFoundError:
        pass
    print("Email ou senha incorretos!")
    return False

def ler_produtos():
    produtos = []
    try:
        with open(PRODUCTS_FILE, 'r', encoding='utf-8') as p:
            for line in p:
                parts = line.strip().split(',', 1)
                if len(parts) != 2:
                    continue
                nome, quantidade = parts
                try:
                    quantidade = int(quantidade)
                except ValueError:
                    quantidade = 0
                produtos.append([nome, quantidade])
    except FileNotFoundError:
        pass
    return produtos

def salvar_produtos(produtos):
    with open(PRODUCTS_FILE, 'w', encoding='utf-8') as p:
        for nome, quantidade in produtos:
            p.write(f'{nome},{quantidade}\n')

def cadastrar_produto():
    nome = input("Digite o nome do produto: ").strip()
    try:
        quantidade = int(input("Digite a quantidade do produto: ").strip())
    except ValueError:
        print("Quantidade inválida.")
        return
    produtos = ler_produtos()
    for prod in produtos:
        if prod[0].lower() == nome.lower():
            prod[1] += quantidade
            salvar_produtos(produtos)
            print("Quantidade adicionada ao produto existente.")
            return
    produtos.append([nome, quantidade])
    salvar_produtos(produtos)
    print("Produto cadastrado com sucesso!")
#visualizar os produtos
def listar_produtos():
    produtos = ler_produtos()
    if not produtos:
        print("Nenhum produto cadastrado.")
        return
    print("Lista de Produtos:")
    for nome, quantidade in produtos:
        print(f'Nome: {nome}, Quantidade: {quantidade}')
#edição dos produtos
def editar_produto():
    produtos = ler_produtos()
    if not produtos:
        print("Nenhum produto cadastrado.")
        return
    listar_produtos()
    nome = input("Digite o nome do produto que deseja editar: ").strip()
    for prod in produtos:
        if prod[0].lower() == nome.lower():
            novo_nome = input(f"Novo nome (pressione Enter para manter '{prod[0]}'): ").strip()
            if novo_nome:
                prod[0] = novo_nome
            try:
                nova_qtd_input = input(f"Nova quantidade (atual {prod[1]}): ").strip()
                if nova_qtd_input:
                    prod[1] = int(nova_qtd_input)
            except ValueError:
                print("Quantidade inválida. Edição cancelada.")
                return
            salvar_produtos(produtos)
            print("Produto editado com sucesso.")
            return
    print("Produto não encontrado.")
#abre os produtos 
def vender_produto():
    produtos = ler_produtos()
    if not produtos:
        print("Nenhum produto cadastrado.")
        return
    listar_produtos()
    nome = input("Digite o nome do produto que deseja vender: ").strip()
    for prod in produtos:
        if prod[0].lower() == nome.lower():
            try:
                qtd_venda = int(input("Digite a quantidade a vender: ").strip())
            except ValueError:
                print("Quantidade inválida.")
                return
            if qtd_venda <= 0:
                print("Quantidade deve ser maior que zero.")
                return
            if qtd_venda > prod[1]:
                print("Quantidade em estoque insuficiente.")
                return
            prod[1] -= qtd_venda
            if prod[1] <= 0:
                produtos.remove(prod)
                print("Produto esgotado e removido.")
            else:
                print("Venda realizada com sucesso.")
            salvar_produtos(produtos)
            return
    print("Produto não encontrado.")

def menu_produtos():
    while True:
        print("\nO que deseja fazer?")
        print("3- Cadastrar Produto")
        print("4- Listar Produto")
        print("5- Editar Produto")
        print("6- Vender Produto")
        print("7- Sair")
        opcao = input("Digite a opção desejada: ").strip()
        if opcao == '3':
            cadastrar_produto()
        elif opcao == '4':
            listar_produtos()
        elif opcao == '5':
            editar_produto()
        elif opcao == '6':
            vender_produto()
        elif opcao == '7':
            print("Saindo...")
            break
        else:
            print("Opção inválida.")
#retorna ao menu principal
def main():
    ensure_files()
    print("Deseja fazer o que?")
    print("1- Cadastrar-se")
    print("2- Login")
    opcao = input("Digite a opção desejada: ").strip()
    if opcao == '1':
        cadastrar_usuario()
        print("Faça login para acessar os produtos.")
        if login():
            menu_produtos()
    elif opcao == '2':
        if login():
            menu_produtos()
    else:
        print("Opção inválida.")

if __name__ == '__main__':
    main()

