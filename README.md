import pandas as pd

def iyimserlik_olcutu(tablo):
    min_iyimser_maliyet_degeri = tablo.to_numpy().min()
    max_iyimser_kazanc_degeri = tablo.to_numpy().max()

    iyimser_maliyetin_konumu = [(row, col) for row in tablo.index for col in tablo.columns if
                                tablo.at[row, col] == min_iyimser_maliyet_degeri]
    iyimser_kazancin_konumu = [(row, col) for row in tablo.index for col in tablo.columns if
                               tablo.at[row, col] == max_iyimser_kazanc_degeri]

    return iyimser_maliyetin_konumu, iyimser_kazancin_konumu


def kotumserlik_olcutu(tablo):
    max_kotumser_kazanc_degeri = tablo.max(axis=1)
    max_kotumser_maliyet_degeri = tablo.max(axis=1)

    kotumser_kazanc_degeri = max_kotumser_kazanc_degeri.max()
    kotumser_maliyet_degeri = max_kotumser_maliyet_degeri.min()

    kotumser_kazancin_konumu = [(row, col) for row in tablo.index for col in tablo.columns if
                                tablo.at[row, col] == kotumser_kazanc_degeri]
    kotumser_maliyetin_konumu = [(row, col) for row in tablo.index for col in tablo.columns if
                                 tablo.at[row, col] == kotumser_maliyet_degeri]
    return kotumser_maliyetin_konumu, kotumser_kazancin_konumu


def hurwics_olcutu(tablo, alpha):
    hurwicz_sonuclari = []

    for row in tablo.index:
        kazanc = alpha * tablo.loc[row].max() + (1 - alpha) * tablo.loc[row].min()
        maliyet = alpha * tablo.loc[row].min() + (1 - alpha) * tablo.loc[row].max()
        hurwicz_sonuclari.append((row, kazanc, maliyet))

    en_iyi_kazanc_index = max(range(len(hurwicz_sonuclari)), key=lambda i: hurwicz_sonuclari[i][1])
    en_iyi_maliyet_index = min(range(len(hurwicz_sonuclari)), key=lambda i: hurwicz_sonuclari[i][2])

    return hurwicz_sonuclari[en_iyi_kazanc_index], hurwicz_sonuclari[en_iyi_maliyet_index]


def laplace_olcutu(tablo):
    satir_ortalamalari = tablo.sum(axis=1) / len(tablo.columns)

    laplace_kazanc_degeri = satir_ortalamalari.max()
    laplace_maliyet_degeri = satir_ortalamalari.min()

    laplace_kazancin_konumu = [(row, col) for row in tablo.index for col in tablo.columns if
                             satir_ortalamalari[row] == laplace_kazanc_degeri]
    laplace_maliyet_konumu = [(row, col) for row in tablo.index for col in tablo.columns if
                              satir_ortalamalari[row] == laplace_maliyet_degeri]

    return laplace_maliyet_konumu, laplace_kazancin_konumu


def beklenen_firsat_kaybi_matrisi(tablo):
    satir_sayisi, sutun_sayisi = tablo.shape
    satir_ortalamalari = tablo.mean(axis=1)

    beklenen_firsat_kaybi_matrisi = pd.DataFrame(index=tablo.index, columns=tablo.columns, dtype=float)

    for i in range(satir_sayisi):
        beklenen_firsat_kaybi_matrisi.iloc[i, :] = satir_ortalamalari.iloc[i]
    return beklenen_firsat_kaybi_matrisi



a = int(input("Satır Sayısını Giriniz: "))
b = int(input("Sütun Sayısını Giriniz: "))
Satir_adlari = [input(f"{i + 1}. Satırın adını Giriniz: ") for i in range(a)]
Sutun_adlari = [input(f"{j + 1}. Sütunun adını giriniz: ") for j in range(b)]



tablo = [[0] * b for _ in range(a)]



for i in range(a):
    for j in range(b):
        tablo[i][j] = int(input(f"{i + 1}. Satır {j + 1}. Sütunun Değerini Giriniz: "))





df = pd.DataFrame(tablo, columns=Sutun_adlari, index=Satir_adlari, dtype=int)
print(df)




iyimser_maliyetin_konumu, iyimser_kazancin_konumu = iyimserlik_olcutu(df)
kotumser_maliyetin_konumu, kotumser_kazancin_konumu = kotumserlik_olcutu(df)



alpha = float(input("iyimserlik Katsayısını Giriniz (0 ile 1 Arasında): "))
hurwicz_kazanc, hurwicz_maliyet = hurwics_olcutu(df, alpha)



#Sonuçları yazdır.
print("\nHurwicz Ölçütü:")
print(f"Kazanç: Satır: {hurwicz_kazanc[0]}, Değer: {hurwicz_kazanc[1]}")
print(f"Maliyet: Satır: {hurwicz_maliyet[0]}, Değer: {hurwicz_maliyet[2]}")

laplace_maliyet_konumu, laplace_kazanc_konumu = laplace_olcutu(df)
optimal_laplace_maliyet_konumu = max(laplace_maliyet_konumu, key=lambda x: df.at[x[0], x[1]])
optimal_laplace_kazanc_konumu = min(laplace_kazanc_konumu, key=lambda x: df.at[x[0], x[1]])

beklenen_firsat_kaybi_matrisi = beklenen_firsat_kaybi_matrisi(df)

print("\nİyimser Maliyet Kriteri:")
for index in iyimser_maliyetin_konumu:
    print(f"Satır: {index[0]}, Sütun: {index[1]}, Değer: {df.at[index[0], index[1]]}")

print("\nİyimser Kazanç Kriteri:")
for index in iyimser_kazancin_konumu:
    print(f"Satır: {index[0]}, Sütun: {index[1]}, Değer: {df.at[index[0], index[1]]}")

print("\nKötümser Maliyet Kriteri:")
for index in kotumser_maliyetin_konumu:
    print(f"Satır: {index[0]}, Sütun: {index[1]}, Değer: {df.at[index[0], index[1]]}")

print("\nKötümser Kazanç Kriteri:")
for index in kotumser_kazancin_konumu:
    print(f"Satır: {index[0]}, Sütun: {index[1]}, Değer: {df.at[index[0], index[1]]}")

print("\nLaplace Kriterine Göre Maliyet:")
print(f"Satır: {optimal_laplace_maliyet_konumu[0]}, Sütun: {optimal_laplace_maliyet_konumu[1]}, Değer: {df.at[optimal_laplace_maliyet_konumu[0], optimal_laplace_maliyet_konumu[1]]}")

print("\nLaplace Kriterine Göre Kazanç:")
print(f"Satır: {optimal_laplace_kazanc_konumu[0]}, Sütun: {optimal_laplace_kazanc_konumu[1]}, Değer: {df.at[optimal_laplace_kazanc_konumu[0], optimal_laplace_kazanc_konumu[1]]}")

print("\nBeklenen Fırsat Kaybı Matrisi:")
print(beklenen_firsat_kaybi_matrisi)

