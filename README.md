from tkinter import *
from tkinter import ttk
import sqlite3

janela =Tk()

class funcs():
    def conectar_banco(self):
        self.conn= sqlite3.connect("clientes.bd")
        self.cursor = self.conn.cursor()
    def desconectar_banco(self):
        self.conn.close()
    def limpar_a_tela(self):
        self.entrada_nome.delete(0, END)
        self.entrada_produto.delete(0, END)
        self.entrada_valor.delete(0, END)
        self.entrada_quantidade.delete(0, END)
        self.entrada_codigo.delete(0, END)
    def variaveis(self):
        self.codigo =self.entrada_codigo.get()
        self.nome = self.entrada_nome.get()
        self.produto = self.entrada_produto.get()
        self.valor = self.entrada_valor.get()
        self.quantidade = self.entrada_quantidade.get()

    def montarTabelas(self):
        self.conectar_banco()
        ### Criando tabela
        self.cursor.execute("""
            CREATE TABLE IF NOT EXISTS cliente(
                codigo INTEGER PRIMARY KEY,
                nome_cliente CHAR(40) NOT NULL,
                produto CHAR(15),
                valor INTEGER(10),
                quantidade INTEGER(10)
            );    
        """)
        self.conn.commit();
        print("Banco de dados criado")
        self.desconectar_banco()
        
    def adicionar_cliente(self):
        self.variaveis()
        self.conectar_banco()
        self.cursor.execute(""" INSERT INTO cliente (nome_cliente, produto, valor, quantidade)
            VALUES (?, ?, ?, ?)""", (self.nome ,self.produto,self.valor, self.quantidade))
        self.conn.commit()
        self.desconectar_banco()
        self.selecionar_na_lista()
        self.limpar_a_tela()
        
    def selecionar_na_lista(self):
        self.listaCli.delete(*self.listaCli.get_children()) ##limpar oque tem na lista
        self.conectar_banco()
        lista = self.cursor.execute("""SELECT codigo , nome_cliente, produto, valor, quantidade FROM cliente
            ORDER BY nome_cliente ASC; """)
        for i in lista:
            self.listaCli.insert("",END, values = i)
        self.desconectar_banco()

    def duplo_click(self, event):
        self.limpar_a_tela()
        self.listaCli.selection()
        for n in self.listaCli.selection():
            col1, col2, col3, col4, col5 = self.listaCli.item(n, 'values')
            self.entrada_codigo.insert(END, col1)
            self.entrada_nome.insert(END, col2)
            self.entrada_produto.insert(END, col3)
            self.entrada_valor.insert(END, col4)
            self.entrada_quantidade.insert(END, col5)

    def apagar_cliente(self):
        self.variaveis()
        self.conectar_banco()
        self.cursor.execute(""" DELETE FROM cliente WHERE codigo = ? """, (self.codigo))
        self.conn.commit()
        self.desconectar_banco()
        self.limpar_a_tela()
        self.selecionar_na_lista()

    def alterar_cliente(self):
        self.variaveis()
        self.conectar_banco()
        self.cursor.execute("""UPDATE cliente SET nome_cliente = ?, produto = ?, valor = ?, quantidade = ?
            WHERE codigo = ?""", (self.nome,self.produto,self.valor,self.quantidade, self.codigo))
        self.conn.commit()
        self.desconectar_banco()
        self.selecionar_na_lista()
        self.limpar_a_tela()

    def buscar_cliente(self):
        self.variaveis()
        self.conectar_banco()
        self.listaCli.delete(*self.listaCli.get_children())
        self.entrada_nome.insert(END, '%')
        nome = self.entrada_nome.get()
        self.cursor.execute("""SELECT codigo, nome_cliente, produto, valor, quantidade FROM cliente
                            WHERE nome_cliente LIKE '%s' ORDER BY nome_cliente ASC """ % nome)
        buscarnomeCli = self.cursor.fetchall()
        for i in buscarnomeCli:
            self.listaCli.insert("", END, values = i)
        self.limpar_a_tela()
        self.desconectar_banco()

class aplicacoes(funcs):
    def __init__(self):
        self.janela = janela
        self.tela()
        self.frames_da_tela()
        self.botoes_de_cima()
        self.botoes_de_baixo()
        self.montarTabelas()
        self.selecionar_na_lista()
        janela.mainloop()
    def tela(self):
        self.janela.title("Ferragens do seu Zé")
        self.janela.configure(background="black")
        self.janela.geometry("900x700")
        self.janela.resizable(True, True)
        self.janela.maxsize(width=1200, height=1000)
        self.janela.minsize(width=800, height=600)
    def frames_da_tela(self):
        self.frame_cima = Frame(self.janela, bd=4, bg='#dfe3ee', highlightbackground='#759fe6',
                                highlightthickness=3)
        self.frame_cima.place(relx=0.02, rely=0.02, relwidth=0.96, relheight=0.46)
        self.frame_baixo = Frame(self.janela, bd=4, bg='#dfe3ee', highlightbackground='#759fe6',
                                 highlightthickness=3)
        self.frame_baixo.place(relx=0.02, rely=0.5, relwidth=0.96, relheight=0.46)
    def botoes_de_cima(self):
        ## Botão "Limpar"
        self.limpar = Button(self.frame_cima, text="Limpar", bd=2, bg="snow", fg="black",
                             font=("verdana", 8, 'bold'), command=self.limpar_a_tela)
        self.limpar.place(relx=0.9, rely=0.32, relheight=0.1, relwidth=0.1)
        ## Botão "Buscar"
        self.buscar = Button(self.frame_cima, text="Buscar", bd=2, bg="snow", fg="black",
                             font=("verdana", 8, 'bold'), command= self.buscar_cliente)
        self.buscar.place(relx=0.79, rely=0.21, relheight=0.1, relwidth=0.10)
        ## Botão "Alterar"
        self.alterar = Button(self.frame_cima, text="Alterar", bd=2, bg="snow", fg="black",
                              font=("verdana", 8, 'bold'), command=self.alterar_cliente)
        self.alterar.place(relx=0.9, rely=0.21, relheight=0.1, relwidth=0.10)
        ## Botão "Novo"
        self.novo = Button(self.frame_cima, text="Novo", bd=2, bg="snow", fg="black",
                           font=("verdana", 8, 'bold'), command= self.adicionar_cliente)
        self.novo.place(relx=0.79, rely=0.1, relheight=0.1, relwidth=0.10)
        ## Botão "Apagar"
        self.apagar = Button(self.frame_cima, text="Apagar", bd=2, bg="snow", fg="black",
                             font=("verdana", 8, 'bold'), command=self.apagar_cliente)
        self.apagar.place(relx=0.9, rely=0.1, relheight=0.1, relwidth=0.10)

        ##Label do nome
        self.lb_nome = Label(self.janela, text="Nome:", bg='#dfe3ee', font=("Arial", 9, 'bold'))
        self.lb_nome.place(relx=0.08, rely=0.03)

        ##Entry do nome
        self.entrada_nome = Entry(self.frame_cima)
        self.entrada_nome.place(relx=0.05, rely=0.10, relwidth=0.68)

        ##Label do produto
        self.lb_produto = Label(self.janela, text="Produto:", bg='#dfe3ee', font=("Arial", 9, 'bold'))
        self.lb_produto.place(relx=0.08, rely=0.13)

        ##Entry do produto
        self.entrada_produto = Entry(self.frame_cima)
        self.entrada_produto.place(relx=0.05, rely=0.32, relwidth=0.68)

        ##Label do valor
        self.lb_valor = Label(self.janela, text="Valor", bg='#dfe3ee', font=("Arial", 9, 'bold'))
        self.lb_valor.place(relx=0.08, rely=0.22)

        ##Entry do valor
        self.entrada_valor = Entry(self.frame_cima)
        self.entrada_valor.place(relx=0.05, rely=0.54, relwidth=0.30)

        ##Label da itens
        self.lb_quantidade = Label(self.janela, text="Quantidade:", bg='#dfe3ee', font=("Arial", 9, 'bold'))
        self.lb_quantidade.place(relx=0.43, rely=0.22)

        ##Entry da itens
        self.entrada_quantidade = Entry(self.frame_cima)
        self.entrada_quantidade.place(relx=0.43, rely=0.54, relwidth=0.30)

        self.lb_codigo = Label(self.janela, text="Código do Cliente:", bg='#dfe3ee', font=("Arial", 9, 'bold'))
        self.lb_codigo.place(relx=0.08, rely=0.33)

        ##Entry do valor
        self.entrada_codigo = Entry(self.frame_cima)
        self.entrada_codigo.place(relx=0.05, rely=0.80, relwidth=0.30)
        
    def botoes_de_baixo(self):
        self.listaCli = ttk.Treeview(self.frame_baixo, height=3, columns=("col1, col2, col3, col4, col5"))
        self.listaCli.heading("#0", text="")
        self.listaCli.heading("#1", text="Código")
        self.listaCli.heading("#2", text="Nome")
        self.listaCli.heading("#3", text="Produto")
        self.listaCli.heading("#4", text="Valor")
        self.listaCli.heading("#5", text="Quantidade")
        # Proporção de 500 para ordenar os itens
        self.listaCli.column("#0", width=1)
        self.listaCli.column("#1", width=50)
        self.listaCli.column("#2", width=200)
        self.listaCli.column("#3", width=100)
        self.listaCli.column("#4", width=100)
        self.listaCli.column("#5", width=50)

        self.listaCli.place(relx=0.01, rely=0.1, relwidth=0.95, relheight=0.86, )

        self.scrool = Scrollbar(self.janela, orient="vertical")
        self.listaCli.configure(yscroll=self.scrool.set)
        self.scrool.place(relx=0.94, rely=0.55, relwidth=0.03, relheight=0.38)
        self.listaCli.bind("<Double-1>", self.duplo_click)

aplicacoes()
