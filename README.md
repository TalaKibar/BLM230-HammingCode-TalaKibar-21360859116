# BLM230-HammingCode-TalaKibar-21360859116
import tkinter as tk
from tkinter import messagebox, simpledialog, ttk

class HammingSimulator:
    def __init__(self, root):
        self.root = root
        self.root.title("Hamming SEC-DED Simülatörü (8/16/32-bit)")
        self.root.geometry("700x500")
        
        self.memory = []
        self.bit_length = tk.IntVar(value=8)
        
        self.create_widgets()
        self.setup_style()

    def setup_style(self):
        style = ttk.Style()
        style.configure('TFrame', background='#f0f0f0')
        style.configure('TLabel', background='#f0f0f0', font=('Arial', 10))
        style.configure('TButton', font=('Arial', 10), padding=5)
        style.configure('TCombobox', font=('Arial', 10))
        style.map('TButton', 
                 foreground=[('active', 'black'), ('!disabled', 'black')],
                 background=[('active', '#d9d9d9'), ('!disabled', '#f0f0f0')])

    def create_widgets(self):
        # Ana frame
        main_frame = ttk.Frame(self.root, padding="10")
        main_frame.pack(fill=tk.BOTH, expand=True)
        
        # Giriş paneli
        input_frame = ttk.LabelFrame(main_frame, text="Giriş", padding="10")
        input_frame.pack(fill=tk.X, pady=5)
        
        # Bit uzunluğu seçimi
        ttk.Label(input_frame, text="Veri Boyutu:").grid(row=0, column=0, padx=5, pady=5, sticky=tk.W)
        bit_menu = ttk.Combobox(input_frame, textvariable=self.bit_length, 
                              values=[8, 16, 32], state='readonly', width=5)
        bit_menu.grid(row=0, column=1, padx=5, pady=5)
        bit_menu.bind("<<ComboboxSelected>>", self.update_input_validation)
        
        # Veri girişi
        ttk.Label(input_frame, text="Veri (binary):").grid(row=1, column=0, padx=5, pady=5, sticky=tk.W)
        self.data_entry = ttk.Entry(input_frame, width=40, font=('Courier New', 10))
        self.data_entry.grid(row=1, column=1, columnspan=2, padx=5, pady=5)
        
        # Butonlar
        btn_frame = ttk.Frame(input_frame)
        btn_frame.grid(row=2, column=0, columnspan=3, pady=10)
        
        ttk.Button(btn_frame, text="Belleğe Yaz", command=self.write_memory).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Hata Ekle", command=self.introduce_error).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Düzelt", command=self.correct_error).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Temizle", command=self.clear).pack(side=tk.LEFT, padx=5)
        
        # Çıktı paneli
        output_frame = ttk.LabelFrame(main_frame, text="Sonuçlar", padding="10")
        output_frame.pack(fill=tk.BOTH, expand=True, pady=5)
        
        # Kaydırma çubuğu ile çıktı alanı
        self.output_text = tk.Text(output_frame, width=80, height=15, 
                                 font=('Courier New', 10), wrap=tk.WORD)
        scrollbar = ttk.Scrollbar(output_frame, orient=tk.VERTICAL, command=self.output_text.yview)
        self.output_text.configure(yscrollcommand=scrollbar.set)
        
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        self.output_text.pack(fill=tk.BOTH, expand=True)
        
        # Durum çubuğu
        self.status_var = tk.StringVar()
        self.status_var.set("Hazır")
        status_bar = ttk.Label(main_frame, textvariable=self.status_var, relief=tk.SUNKEN)
        status_bar.pack(fill=tk.X, pady=(5,0))

    def update_input_validation(self, event=None):
        # Giriş alanını temizle
        self.data_entry.delete(0, tk.END)
        self.status_var.set(f"{self.bit_length.get()}-bit modu seçildi. Lütfen {self.bit_length.get()} bitlik veri girin.")

    def write_memory(self):
        data = self.data_entry.get().strip()
        bit_size = self.bit_length.get()
        
        if len(data) != bit_size:
            messagebox.showerror("Hata", f"{bit_size}-bit modu için veri uzunluğu {bit_size} bit olmalıdır!")
            return
            
        if not all(c in '01' for c in data):
            messagebox.showerror("Hata", "Sadece 0 ve 1 girebilirsiniz!")
            return

        encoded = self.hamming_encode(data, bit_size)
        self.memory.append(encoded)
        
        # Hamming kod detaylarını hesapla
        m = bit_size
        r = len(encoded) - m - 1  # Genel parity hariç parity bit sayısı
        
        self.output_text.insert(tk.END, f"\n[8-bit Veri] {data}\n")
        self.output_text.insert(tk.END, f"[Hamming Kodu] {encoded} (Toplam {len(encoded)} bit)\n")
        self.output_text.insert(tk.END, f" - Veri bitleri: {m}\n")
        self.output_text.insert(tk.END, f" - Parity bitleri: {r} + 1 (genel parity)\n")
        
        self.status_var.set(f"{bit_size}-bit veri başarıyla belleğe yazıldı. Toplam bellek: {len(self.memory)} veri")

    def introduce_error(self):
        if not self.memory:
            messagebox.showerror("Hata", "Bellekte veri yok!")
            return
            
        last_code = self.memory[-1]
        bit_size = self.bit_length.get()
        
        # Hangi bitin bozulacağını sor
        bit_pos = simpledialog.askinteger("Hata Ekle", 
                                        f"Hangi bit bozulacak? (0-{len(last_code)-1} arası)", 
                                        parent=self.root,
                                        minvalue=0, 
                                        maxvalue=len(last_code)-1)
        
        if bit_pos is None:  # Kullanıcı iptal etti
            return
            
        code_list = list(last_code)
        code_list[bit_pos] = '1' if code_list[bit_pos] == '0' else '0'
        self.memory[-1] = ''.join(code_list)
        
        self.output_text.insert(tk.END, f"\n[YAPAY HATA] Bit {bit_pos} değiştirildi:\n")
        self.output_text.insert(tk.END, f"Bozulmuş Kod: {self.memory[-1]}\n")
        
        # Hata bitinin türünü belirle (veri/parity)
        if self.is_power_of_two(bit_pos+1) or bit_pos == 0:
            bit_type = "Parity biti"
        else:
            bit_type = "Veri biti"
            
        self.output_text.insert(tk.END, f" - {bit_type} değiştirildi\n")
        self.status_var.set(f"Bit {bit_pos} başarıyla bozuldu ({bit_type})")

    def correct_error(self):
        if not self.memory:
            messagebox.showerror("Hata", "Bellekte veri yok!")
            return
            
        encoded = self.memory[-1]
        bit_size = self.bit_length.get()
        corrected, position, error_type = self.hamming_decode(encoded, bit_size)
        
        self.output_text.insert(tk.END, "\n[HATA ANALİZİ]\n")
        
        if position == -1 and error_type == "none":
            self.output_text.insert(tk.END, " - Hata yok, veri doğru\n")
        elif error_type == "single":
            self.memory[-1] = corrected  # Belleği güncelle
            self.output_text.insert(tk.END, f" - Tek bit hatası bulundu (Bit {position})\n")
            self.output_text.insert(tk.END, f" - Düzeltilmiş Kod: {corrected}\n")
        elif error_type == "double":
            self.output_text.insert(tk.END, " - Çift bit hatası tespit edildi (Düzeltilemez!)\n")
        
        self.status_var.set(f"Hata analizi tamamlandı: {error_type} hata")

    def clear(self):
        self.output_text.delete('1.0', tk.END)
        self.status_var.set("Çıktı temizlendi")

    def hamming_encode(self, data, bit_size):
        m = len(data)
        r = 0
        
        # Gerekli parity bit sayısını hesapla
        while (2 ** r) < (m + r + 1):
            r += 1
            
        total_len = m + r + 1  # +1 for overall parity
        code = ['0'] * total_len
        
        # Veri bitlerini yerleştir (parity bitlerini atlayarak)
        data_pos = 0
        for i in range(1, total_len):
            if not self.is_power_of_two(i):
                code[i] = data[data_pos]
                data_pos += 1
        
        # Parity bitlerini hesapla
        for i in range(r):
            parity_pos = 2 ** i
            parity = 0
            for j in range(1, total_len):
                if j & parity_pos:
                    parity ^= int(code[j])
            code[parity_pos] = str(parity)
        
        # Genel parity bitini hesapla (DED için)
        overall_parity = sum(int(x) for x in code[1:]) % 2
        code[0] = str(overall_parity)
        
        return ''.join(code)

    def hamming_decode(self, code, bit_size):
        code = list(code)
        m = bit_size
        r = 0
        
        while (2 ** r) < (m + r + 1):
            r += 1
            
        # Genel parity kontrolü (DED)
        overall_parity = sum(int(x) for x in code) % 2
        
        # Sendrom hesapla (SEC)
        syndrome = 0
        for i in range(r):
            parity_pos = 2 ** i
            parity = 0
            for j in range(1, len(code)):
                if j & parity_pos:
                    parity ^= int(code[j])
            if parity:
                syndrome += parity_pos
        
        # Hata analizi
        if syndrome == 0:
            if overall_parity == 0:
                return (''.join(code), -1, "none")  # Hata yok
            else:
                return (''.join(code), -1, "double")  # Çift bit hatası
        else:
            if overall_parity == 1:
                # Tek bit hatası - düzelt
                if syndrome < len(code):
                    code[syndrome] = '1' if code[syndrome] == '0' else '0'
                return (''.join(code), syndrome, "single")
            else:
                # Çift bit hatası - düzeltilemez
                return (''.join(code), -1, "double")

    def is_power_of_two(self, x):
        return x != 0 and (x & (x-1)) == 0


if __name__ == '__main__':
    root = tk.Tk()
    app = HammingSimulator(root)
    root.mainloop()
