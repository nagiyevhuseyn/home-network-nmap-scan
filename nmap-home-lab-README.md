# Ev Şəbəkəsi Təhlükəsizlik Skan Layihəsi (Nmap)

## Məqsəd

VirtualBox üzərində Ubuntu virtual maşını qurub, Nmap alətindən istifadə edərək
yerli şəbəkədəki cihazları aşkarlamaq və açıq portları/servisləri analiz etmək.
Məqsəd — real SOC/şəbəkə administrasiyası işlərində istifadə olunan əsas
kəşfiyyat (reconnaissance) prosesini əməli şəkildə öyrənmək.

## İstifadə olunan mühit

- **Virtuallaşdırma:** VirtualBox
- **Əməliyyat sistemi:** Ubuntu (Guest OS)
- **Şəbəkə rejimi:** NAT (VirtualBox default)
- **Alət:** Nmap

## Addımlar

### 1. VM-in öz IP ünvanının tapılması

```bash
ip a
```

Nəticə: VM-in IP ünvanı `10.0.2.15` olaraq müəyyən edildi (VirtualBox NAT
şəbəkəsinin standart aralığı).

### 2. Şəbəkədəki aktiv cihazların aşkarlanması

```bash
sudo nmap -sn 10.0.2.0/24
```

Nəticədə 3 aktiv host tapıldı:
- `10.0.2.2` — VirtualBox NAT gateway (host kompüterin virtual şəbəkə çıxışı)
- `10.0.2.3` — VirtualBox-un daxili DNS serveri
- `10.0.2.15` — Ubuntu VM-in özü

### 3. Gateway-in (10.0.2.2) detallı skan edilməsi

```bash
sudo nmap -sV 10.0.2.2
```

**Nəticə:**

```
Nmap scan report for _gateway (10.0.2.2)
PORT     STATE SERVICE
135/tcp  open  msrpc
445/tcp  open  microsoft-ds
5357/tcp open  wsdapi
```

## Tapıntıların Analizi

| Port | Servis | İzah | Risk qiymətləndirməsi |
|------|--------|------|------------------------|
| 135  | MSRPC (Microsoft RPC) | Windows-un daxili proseslərarası əlaqə xidməti | Yerli şəbəkədə normal, amma internetə açıq olarsa uzaqdan icra hücumlarına şərait yarada bilər |
| 445  | SMB (microsoft-ds) | Fayl/printer paylaşımı üçün istifadə olunan protokol | Tarixən ən çox hədəflənən portlardan biri (məs. WannaCry ransomware bu porta hücum edib). İnternetə açıq olmamalıdır |
| 5357 | WSDAPI | Şəbəkə cihazlarının avtomatik kəşfi xidməti | Adətən aşağı risk, amma lazım deyilsə bağlanması tövsiyə olunur |

## Nəticə və Tövsiyələr

Bu skan, host kompüterin (Windows) VirtualBox-un NAT şəbəkəsi vasitəsilə necə
göründüyünü nümayiş etdirdi. Real mühitdə bu cür portların açıq olması aşağıdakı
tövsiyələri doğurur:

1. **445 (SMB)** portu heç vaxt birbaşa internetə açıq saxlanılmamalıdır —
   yalnız etibarlı yerli şəbəkə daxilində istifadə olunmalıdır.
2. Lazım olmayan servislər (məs. WSDAPI istifadə olunmursa) söndürülməlidir —
   bu, "attack surface" (hücum səthini) azaldır.
3. Şəbəkə seqmentasiyası (VLAN-lar) tətbiq etməklə, kritik xidmətlər ictimai
   şəbəkədən ayrıla bilər.

## Öyrənilən əsas anlayışlar

- Nmap ilə host discovery (`-sn`) və service/version detection (`-sV`) fərqi
- CIDR notation (`/24`) və şəbəkə aralığının hesablanması
- Açıq portların real dünyada yaratdığı təhlükəsizlik riskləri
- VirtualBox NAT şəbəkəsinin necə işlədiyi

## Qeyd

Bu skan yalnız öz şəxsi VirtualBox mühitimdə, öz icazəm daxilində aparılmışdır.
İcazəsiz şəbəkə skanı etmək qanunsuzdur və edilməməlidir.
