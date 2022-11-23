---
author: "rand0m"
title: "セキュリティー・キャンプ / [C1]痕跡から手がかりを集める のWriteupっぽい何か"
date: "2022-08-15"
description: "A writeupish something for the forensics class of SECCAMP 2022"
tags: ["security camp", "security camp 22"]
categories: ["security camp"]
---

[2022-08-08から2022-08-12に開催されたセキュリティー・キャンプ](https://www.ipa.go.jp/jinzai/camp/2022/zenkoku2022_index.html)に参加しました。「[C1] 痕跡から手がかりを集める - アーティファクトの分析」のwriteupです。(なお、解いた問題のflagをメモするのを忘れてしまったので、復習した問題のwriteupです(???))

<!--more-->

結果: ![result](https://user-images.githubusercontent.com/54098069/184836443-8d47807b-ef40-47d5-a0a5-75aa345429a3.png)

## Excercise
**シナリオ:**
- 2022-06-12 18:18:41 (JST): 不審な通信をSOCサービスが検知
- コンピュータ名: `L1458`
  - OS: Windows 10
  - IPアドレス: `172.30.42.53`
  - ユーザ名: `sryoma77`
  - 実際にされた対応: CFOの利用端末なのでシャットダウン
  - 方針: イベントログを調査
- コンピュータ名: `L2807`
  - OS: Windows 10
  - IPアドレス: `172.30.42.107`
  - ユーザ名: `shizriku`
  - 説明: 他の不審なファイルが存在 / イベントログが消去された形跡が存在
  - 方針: **メモリ・ディスクフォレンジックス調査**
- コンピュータ名: `O4347`
  - OS: Windows 11
  - IPアドレス: `172.30.42.84`
  - ユーザ名: `mori2sei`
  - 方針: イベントログを調査
  
他にもあったのですが、書くと長くなるので省略します。email用のサブドメインもあり作り込みがすごいなと思いました。

**与えられた解析ファイル**
- `l2807_C1-001.e01` (sha256: `1668859f11fdff61e9d16bbf718ff932883eba2d009e4080a949047b96739eea`)

-----

### Hash
怪しい実行ファイルのハッシュをVTで検索するという問題でした。(省略)
### Persistance
| Auto-Start Extensibility Points                                     | 解析結果                                                        |
| :------------------------------------------------------------------ | :-------------------------------------------------------------- |
| [Active Setup](https://attack.mitre.org/techniques/T1547/014/)      | なし                                                            |
| [Registry Run keys](https://attack.mitre.org/techniques/T1547/001/) | ユーザーログイン時に`diagmonu.exe`の実行                        |
| [Scheduled Tasks](https://attack.mitre.org/techniques/T1053/005/)   | [Microsoft Defenderの無効化/ldr_ie.exeの実行](#scheduled-tasks) |
| [Services](https://attack.mitre.org/techniques/T1543/003/)          | [`ldr_od.exe`の実行](#永続化3)                                  |
| [Startup Programs](https://attack.mitre.org/techniques/T1547/001/)  | なし                                                            |
| [WinLogon Keys](https://attack.mitre.org/techniques/T1547/004/)     | なし                                                            |

#### Scheduled Tasks
![ST](/CTF2022i/ST.png)


#### 永続化(3)
> `C:\Users\Public\Documents`フォルダ内のexeファイルがサービスとして起動するように設定されていました。キーのタイムスタンプから設定されたと判断できる日時をYYYY-MM-DD hh:mm:ddの形式で、日本時間(JST)で回答してください。

Servicesを確認する様子．
![ldr_od.exeがservicesに登録されている様子](/CTF2022i/ldr.png)
`ldr_od.exe`のtimestamp 2022-06-12 05:06:14 UTCをJSTに変換すれば良い．

Flag: `2022-06-12 14:06:14`
### タイムスタンプ
(省略)
### SRUM
System Resource Usage Monitor (SRUM)を分析する問題。`C:\Windows\System32\sru\SRUDB.dat`と`C:\Windows\System32\config\SOFTWARE`を[`SRUM-DUMP2`](https://github.com/MarkBaggett/srum-dump)を用いて解析する。特にNetwork Data UsageやApplication Resource Usageを見た。

#### SRUM(4)
> プログラムの実行ユーザ(User SID)に注目すると、C:\Users\Public\Documents配下へのexeファイルの設置以降で、定常的にSYSTEM権限(S-1-5-18)で動作しているプログラムがありました。このプログラムのファイル名を回答してください。

![SYSTEM権限(S-1-5-18)でフィルターした結果](/CTF2022i/15f6b4c6edc89fe6ca3803cb6af04d7fafb762e9ccdf9407caadfed3.png)
SYSTEM権限(S-1-5-18)でフィルターした結果`iexplore.exe`が多く見られた。

Flag: `iexplore.exe`

### Prefetch
#### プリフェッチ(4)
> powershell.exeのPFファイルの分析結果から、これまでに判明していなかった不審なファイルの存在を確認することができました。 ファイル名をパス情報を含めて回答してください。

以下`C:\Users\Public\Documents`以下のファイルを見ている画像
![C:\Users\Public\Documents以下のファイル](/CTF2022i/2d36d8f6271187427c18f1c6b7b2722942f44ce20fb5bac85543c370.png)
以下、WinPrefetchViewで`POWERSHELL.EXE`に関するプリフェッチファイルを見ている様子。

`POWERSHELL.EXE-022A1004.pf`に`\USERS\SHIZRIKU.RETRICKS\DOWNLOADS\コンサルティング要件定義書\コンサルティング要件定義書.LNK`が存在する．Powershellが日本語を含むLNKファイルを扱うのは不自然だと考えた．
参考: [[MS-SHLLINK]: Shell Link (.LNK) Binary File Format](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-shllink/16cb4ca1-9339-4d0c-a68d-bf1d6cc0f943)

![](/CTF2022i/sus_LNK.png)

Flag: `\USERS\SHIZRIKU.RETRICKS\DOWNLOADS\コンサルティング要件定義書\コンサルティング要件定義書.LNK`

### Analysis
#### Analysis(6)
> shizrikuユーザのNTLMパスワードハッシュを回答してください。

攻撃者は実行ファイル(`mi64.exe`)だけでなく、[Mimikatz](https://github.com/gentilkiwi/mimikatz)のログ(`mi.txt`)をも残している。`mi.txt`を読むと、
```
mimikatz(commandline) # sekurlsa::logonpasswords

Authentication Id : 0 ; 350811 (00000000:00055a5b)
Session           : Interactive from 1
User Name         : shizriku
...(省略)...
Logon Time        : 2022/06/12 14:11:00
SID               : S-1-5-21-3704684023-795799290-1483735124-1001
	msv :	
	 [00000003] Primary
	 * Username : shizriku
	 * Domain   : RETRICKS
	 * NTLM     : a621b48e85488d38790fcbb8520e307e
...(省略)...
```
とある。  
参考: https://jpcertcc.github.io/ToolAnalysisResultSheet_jp/details/Mimikatz_sekurlsa-logonpasswords.htm     

Flag: `a621b48e85488d38790fcbb8520e307e`

