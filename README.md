# RU_to_LATAM-ver-2.0
# tradutor russo offline. Desenvolvi para me ajudar com  estudos.

	import tkinter as tk
	from tkinter import ttk
	from tkinter import scrolledtext
	from tkinter import messagebox
	import threading
	
	# Será necessário instalar esta biblioteca:
	# pip install deep-translator
	try:
	    from deep_translator import GoogleTranslator
	    translator_available = True
	except ImportError:
	    translator_available = False
	
	# Variável global para controlar o modo de saída
	usar_maiusculas = True
	
	def translitera_russo_latino(texto):
	    global usar_maiusculas
	    tabela_transliteracao = {
	        "А": "A", "Б": "B", "В": "V", "Г": "G", "Д": "D",
	        "Е": "E", "Ё": "YO", "Ж": "ZH", "З": "Z", "И": "I",
	        "Й": "I", "К": "K", "Л": "L", "М": "M", "Н": "N",
	        "О": "O", "П": "P", "Р": "R", "С": "S", "Т": "T",
	        "У": "U", "Ф": "F", "Х": "KH", "Ц": "TS", "Ч": "CH",
	        "Ш": "SH", "Щ": "SHCH", "Ъ": "", "Ы": "Y", "Ь": "'",
	        "Э": "E", "Ю": "YU", "Я": "YA"
	    }
	    
	    # Converte cada letra
	    resultado = ''.join([tabela_transliteracao.get(letra.upper(), letra) for letra in texto])
	    
	    # Aplica formatação conforme configuração
	    if usar_maiusculas:
	        return resultado.upper()
	    else:
	        return resultado.lower()
	
	def realizar_transliteracao():
	    texto_russo = entrada_texto.get("1.0", tk.END).strip()
	    texto_latino = translitera_russo_latino(texto_russo)
	    saida_texto_latino.delete("1.0", tk.END)
	    saida_texto_latino.insert("1.0", texto_latino)
	
	def traduzir_texto():
	    if not translator_available:
	        messagebox.showerror("Erro", "Biblioteca de tradução não instalada.\nInstale usando:\npip install deep-translator")
	        return
	    
	    texto_russo = entrada_texto.get("1.0", tk.END).strip()
	    if not texto_russo:
	        messagebox.showinfo("Aviso", "Por favor, insira um texto em russo para traduzir.")
	        return
	    
	    # Atualizar a interface para mostrar que está traduzindo
	    status_label.config(text="Traduzindo... Por favor, aguarde.", foreground="blue")
	    app.update_idletasks()
	    
	    # Executar a tradução em uma thread separada para não bloquear a interface
	    def traduzir_em_thread():
	        try:
	            # Usando deep-translator para fazer a tradução
	            translator = GoogleTranslator(source='ru', target='pt')
	            traducao = translator.translate(texto_russo)
	            
	            # Atualizar a interface na thread principal
	            app.after(0, lambda: saida_texto_ptbr.delete("1.0", tk.END))
	            app.after(0, lambda: saida_texto_ptbr.insert("1.0", traducao))
	            app.after(0, lambda: status_label.config(text="Tradução concluída!", foreground="green"))
	            app.after(2000, lambda: status_label.config(text=""))
	        except Exception as e:
	            app.after(0, lambda: status_label.config(text=f"Erro na tradução: {str(e)}", foreground="red"))
	            app.after(0, lambda: messagebox.showerror("Erro na Tradução", f"Ocorreu um erro: {str(e)}"))
	    
	    # Iniciar a thread de tradução
	    thread = threading.Thread(target=traduzir_em_thread)
	    thread.daemon = True
	    thread.start()
	
	def limpar_campos():
	    entrada_texto.delete("1.0", tk.END)
	    saida_texto_latino.delete("1.0", tk.END)
	    saida_texto_ptbr.delete("1.0", tk.END)
	
	def copiar_resultado_latino():
	    app.clipboard_clear()
	    app.clipboard_append(saida_texto_latino.get("1.0", tk.END).strip())
	    status_label.config(text="Texto transliterado copiado para a área de transferência!", foreground="green")
	    app.after(2000, lambda: status_label.config(text=""))
	
	def copiar_resultado_ptbr():
	    app.clipboard_clear()
	    app.clipboard_append(saida_texto_ptbr.get("1.0", tk.END).strip())
	    status_label.config(text="Texto traduzido copiado para a área de transferência!", foreground="green")
	    app.after(2000, lambda: status_label.config(text=""))
	
	# Função para alternar entre saída em maiúsculas ou minúsculas
	def alternar_maiusculas_minusculas():
	    global usar_maiusculas
	    usar_maiusculas = not usar_maiusculas
	    
	    if usar_maiusculas:
	        botao_alternar.config(text="Usar Minúsculas")
	        status_label.config(text="Saída definida para MAIÚSCULAS", foreground="blue")
	    else:
	        botao_alternar.config(text="Usar Maiúsculas")
	        status_label.config(text="Saída definida para minúsculas", foreground="blue")
	    
	    # Atualiza a saída se já houver texto
	    if saida_texto_latino.get("1.0", tk.END).strip():
	        realizar_transliteracao()
	    
	    app.after(2000, lambda: status_label.config(text=""))
	
	# Configuração da janela principal
	app = tk.Tk()
	app.title("Transliterador e Tradutor Russo")
	app.geometry("900x700")
	app.resizable(True, True)
	
	# Configuração de estilo
	app.configure(bg="#f0f0f0")
	style = ttk.Style()
	style.configure("TButton", padding=6, relief="flat", background="#4CAF50")
	style.configure("TFrame", background="#f0f0f0")
	style.configure("TLabel", background="#f0f0f0", font=("Arial", 11))
	
	# Frame principal
	main_frame = ttk.Frame(app, padding="10")
	main_frame.pack(fill=tk.BOTH, expand=True)
	
	# Título
	titulo = ttk.Label(main_frame, text="Transliterador e Tradutor Russo", font=("Arial", 16, "bold"))
	titulo.pack(pady=10)
	
	# Frame para entrada
	frame_entrada = ttk.Frame(main_frame)
	frame_entrada.pack(fill=tk.BOTH, expand=True, pady=5)
	
	ttk.Label(frame_entrada, text="Texto em Russo:").pack(anchor=tk.W)
	
	entrada_texto = scrolledtext.ScrolledText(frame_entrada, height=6, width=80, font=("Arial", 12))
	entrada_texto.pack(fill=tk.BOTH, expand=True, pady=5)
	
	# Frame para botões
	frame_botoes = ttk.Frame(main_frame)
	frame_botoes.pack(fill=tk.X, pady=10)
	
	botao_transliterar = ttk.Button(
	    frame_botoes, 
	    text="Transliterar",
	    command=realizar_transliteracao
	)
	botao_transliterar.pack(side=tk.LEFT, padx=5)
	
	botao_traduzir = ttk.Button(
	    frame_botoes, 
	    text="Traduzir para PT-BR",
	    command=traduzir_texto
	)
	botao_traduzir.pack(side=tk.LEFT, padx=5)
	
	botao_limpar = ttk.Button(
	    frame_botoes, 
	    text="Limpar Tudo",
	    command=limpar_campos
	)
	botao_limpar.pack(side=tk.LEFT, padx=5)
	
	botao_alternar = ttk.Button(
	    frame_botoes, 
	    text="Usar Minúsculas",
	    command=alternar_maiusculas_minusculas
	)
	botao_alternar.pack(side=tk.LEFT, padx=5)
	
	# Frame para saída transliteração
	frame_saida_latino = ttk.Frame(main_frame)
	frame_saida_latino.pack(fill=tk.BOTH, expand=True, pady=5)
	
	ttk.Label(frame_saida_latino, text="Texto Transliterado (Latino):").pack(anchor=tk.W)
	
	saida_texto_latino = scrolledtext.ScrolledText(frame_saida_latino, height=5, width=80, font=("Arial", 12))
	saida_texto_latino.pack(fill=tk.BOTH, expand=True, pady=5)
	
	botao_copiar_latino = ttk.Button(
	    frame_saida_latino, 
	    text="Copiar Transliteração",
	    command=copiar_resultado_latino
	)
	botao_copiar_latino.pack(anchor=tk.E, padx=5, pady=2)
	
	# Frame para saída tradução
	frame_saida_ptbr = ttk.Frame(main_frame)
	frame_saida_ptbr.pack(fill=tk.BOTH, expand=True, pady=5)
	
	ttk.Label(frame_saida_ptbr, text="Texto Traduzido (Português):").pack(anchor=tk.W)
	
	saida_texto_ptbr = scrolledtext.ScrolledText(frame_saida_ptbr, height=5, width=80, font=("Arial", 12))
	saida_texto_ptbr.pack(fill=tk.BOTH, expand=True, pady=5)
	
	botao_copiar_ptbr = ttk.Button(
	    frame_saida_ptbr, 
	    text="Copiar Tradução",
	    command=copiar_resultado_ptbr
	)
	botao_copiar_ptbr.pack(anchor=tk.E, padx=5, pady=2)
	
	# Status label para feedback
	status_label = ttk.Label(main_frame, text="", foreground="green")
	status_label.pack(pady=5)
	
	# Verificar se a biblioteca está instalada ao iniciar
	if not translator_available:
	    messagebox.showwarning("Aviso", "Biblioteca de tradução não encontrada.\nPara habilitar a tradução, instale:\npip install deep-translator")
	
	# Iniciar a aplicação
	app.mainloop()import tkinter as tk
	from tkinter import ttk
	from tkinter import scrolledtext
	from tkinter import messagebox
	import threading
	
	# Será necessário instalar esta biblioteca:
	# pip install deep-translator
	try:
	    from deep_translator import GoogleTranslator
	    translator_available = True
	except ImportError:
	    translator_available = False
	
	# Variável global para controlar o modo de saída
	usar_maiusculas = True
	
	def translitera_russo_latino(texto):
	    global usar_maiusculas
	    tabela_transliteracao = {
	        "А": "A", "Б": "B", "В": "V", "Г": "G", "Д": "D",
	        "Е": "E", "Ё": "YO", "Ж": "ZH", "З": "Z", "И": "I",
	        "Й": "I", "К": "K", "Л": "L", "М": "M", "Н": "N",
	        "О": "O", "П": "P", "Р": "R", "С": "S", "Т": "T",
	        "У": "U", "Ф": "F", "Х": "KH", "Ц": "TS", "Ч": "CH",
	        "Ш": "SH", "Щ": "SHCH", "Ъ": "", "Ы": "Y", "Ь": "'",
	        "Э": "E", "Ю": "YU", "Я": "YA"
	    }
	    
	    # Converte cada letra
	    resultado = ''.join([tabela_transliteracao.get(letra.upper(), letra) for letra in texto])
	    
	    # Aplica formatação conforme configuração
	    if usar_maiusculas:
	        return resultado.upper()
	    else:
	        return resultado.lower()
	
	def realizar_transliteracao():
	    texto_russo = entrada_texto.get("1.0", tk.END).strip()
	    texto_latino = translitera_russo_latino(texto_russo)
	    saida_texto_latino.delete("1.0", tk.END)
	    saida_texto_latino.insert("1.0", texto_latino)
	
	def traduzir_texto():
	    if not translator_available:
	        messagebox.showerror("Erro", "Biblioteca de tradução não instalada.\nInstale usando:\npip install deep-translator")
	        return
	    
	    texto_russo = entrada_texto.get("1.0", tk.END).strip()
	    if not texto_russo:
	        messagebox.showinfo("Aviso", "Por favor, insira um texto em russo para traduzir.")
	        return
	    
	    # Atualizar a interface para mostrar que está traduzindo
	    status_label.config(text="Traduzindo... Por favor, aguarde.", foreground="blue")
	    app.update_idletasks()
	    
	    # Executar a tradução em uma thread separada para não bloquear a interface
	    def traduzir_em_thread():
	        try:
	            # Usando deep-translator para fazer a tradução
	            translator = GoogleTranslator(source='ru', target='pt')
	            traducao = translator.translate(texto_russo)
	            
	            # Atualizar a interface na thread principal
	            app.after(0, lambda: saida_texto_ptbr.delete("1.0", tk.END))
	            app.after(0, lambda: saida_texto_ptbr.insert("1.0", traducao))
	            app.after(0, lambda: status_label.config(text="Tradução concluída!", foreground="green"))
	            app.after(2000, lambda: status_label.config(text=""))
	        except Exception as e:
	            app.after(0, lambda: status_label.config(text=f"Erro na tradução: {str(e)}", foreground="red"))
	            app.after(0, lambda: messagebox.showerror("Erro na Tradução", f"Ocorreu um erro: {str(e)}"))
	    
	    # Iniciar a thread de tradução
	    thread = threading.Thread(target=traduzir_em_thread)
	    thread.daemon = True
	    thread.start()
	
	def limpar_campos():
	    entrada_texto.delete("1.0", tk.END)
	    saida_texto_latino.delete("1.0", tk.END)
	    saida_texto_ptbr.delete("1.0", tk.END)
	
	def copiar_resultado_latino():
	    app.clipboard_clear()
	    app.clipboard_append(saida_texto_latino.get("1.0", tk.END).strip())
	    status_label.config(text="Texto transliterado copiado para a área de transferência!", foreground="green")
	    app.after(2000, lambda: status_label.config(text=""))
	
	def copiar_resultado_ptbr():
	    app.clipboard_clear()
	    app.clipboard_append(saida_texto_ptbr.get("1.0", tk.END).strip())
	    status_label.config(text="Texto traduzido copiado para a área de transferência!", foreground="green")
	    app.after(2000, lambda: status_label.config(text=""))
	
	# Função para alternar entre saída em maiúsculas ou minúsculas
	def alternar_maiusculas_minusculas():
	    global usar_maiusculas
	    usar_maiusculas = not usar_maiusculas
	    
	    if usar_maiusculas:
	        botao_alternar.config(text="Usar Minúsculas")
	        status_label.config(text="Saída definida para MAIÚSCULAS", foreground="blue")
	    else:
	        botao_alternar.config(text="Usar Maiúsculas")
	        status_label.config(text="Saída definida para minúsculas", foreground="blue")
	    
	    # Atualiza a saída se já houver texto
	    if saida_texto_latino.get("1.0", tk.END).strip():
	        realizar_transliteracao()
	    
	    app.after(2000, lambda: status_label.config(text=""))
	
	# Configuração da janela principal
	app = tk.Tk()
	app.title("Transliterador e Tradutor Russo")
	app.geometry("900x700")
	app.resizable(True, True)
	
	# Configuração de estilo
	app.configure(bg="#f0f0f0")
	style = ttk.Style()
	style.configure("TButton", padding=6, relief="flat", background="#4CAF50")
	style.configure("TFrame", background="#f0f0f0")
	style.configure("TLabel", background="#f0f0f0", font=("Arial", 11))
	
	# Frame principal
	main_frame = ttk.Frame(app, padding="10")
	main_frame.pack(fill=tk.BOTH, expand=True)
	
	# Título
	titulo = ttk.Label(main_frame, text="Transliterador e Tradutor Russo", font=("Arial", 16, "bold"))
	titulo.pack(pady=10)
	
	# Frame para entrada
	frame_entrada = ttk.Frame(main_frame)
	frame_entrada.pack(fill=tk.BOTH, expand=True, pady=5)
	
	ttk.Label(frame_entrada, text="Texto em Russo:").pack(anchor=tk.W)
	
	entrada_texto = scrolledtext.ScrolledText(frame_entrada, height=6, width=80, font=("Arial", 12))
	entrada_texto.pack(fill=tk.BOTH, expand=True, pady=5)
	
	# Frame para botões
	frame_botoes = ttk.Frame(main_frame)
	frame_botoes.pack(fill=tk.X, pady=10)
	
	botao_transliterar = ttk.Button(
	    frame_botoes, 
	    text="Transliterar",
	    command=realizar_transliteracao
	)
	botao_transliterar.pack(side=tk.LEFT, padx=5)
	
	botao_traduzir = ttk.Button(
	    frame_botoes, 
	    text="Traduzir para PT-BR",
	    command=traduzir_texto
	)
	botao_traduzir.pack(side=tk.LEFT, padx=5)
	
	botao_limpar = ttk.Button(
	    frame_botoes, 
	    text="Limpar Tudo",
	    command=limpar_campos
	)
	botao_limpar.pack(side=tk.LEFT, padx=5)
	
	botao_alternar = ttk.Button(
	    frame_botoes, 
	    text="Usar Minúsculas",
	    command=alternar_maiusculas_minusculas
	)
	botao_alternar.pack(side=tk.LEFT, padx=5)
	
	# Frame para saída transliteração
	frame_saida_latino = ttk.Frame(main_frame)
	frame_saida_latino.pack(fill=tk.BOTH, expand=True, pady=5)
	
	ttk.Label(frame_saida_latino, text="Texto Transliterado (Latino):").pack(anchor=tk.W)
	
	saida_texto_latino = scrolledtext.ScrolledText(frame_saida_latino, height=5, width=80, font=("Arial", 12))
	saida_texto_latino.pack(fill=tk.BOTH, expand=True, pady=5)
	
	botao_copiar_latino = ttk.Button(
	    frame_saida_latino, 
	    text="Copiar Transliteração",
	    command=copiar_resultado_latino
	)
	botao_copiar_latino.pack(anchor=tk.E, padx=5, pady=2)
	
	# Frame para saída tradução
	frame_saida_ptbr = ttk.Frame(main_frame)
	frame_saida_ptbr.pack(fill=tk.BOTH, expand=True, pady=5)
	
	ttk.Label(frame_saida_ptbr, text="Texto Traduzido (Português):").pack(anchor=tk.W)
	
	saida_texto_ptbr = scrolledtext.ScrolledText(frame_saida_ptbr, height=5, width=80, font=("Arial", 12))
	saida_texto_ptbr.pack(fill=tk.BOTH, expand=True, pady=5)
	
	botao_copiar_ptbr = ttk.Button(
	    frame_saida_ptbr, 
	    text="Copiar Tradução",
	    command=copiar_resultado_ptbr
	)
	botao_copiar_ptbr.pack(anchor=tk.E, padx=5, pady=2)
	
	# Status label para feedback
	status_label = ttk.Label(main_frame, text="", foreground="green")
	status_label.pack(pady=5)
	
	# Verificar se a biblioteca está instalada ao iniciar
	if not translator_available:
	    messagebox.showwarning("Aviso", "Biblioteca de tradução não encontrada.\nPara habilitar a tradução, instale:\npip install deep-translator")
	
	# Iniciar a aplicação
	app.mainloop()
