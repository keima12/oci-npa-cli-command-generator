# OCI Network Path Analyzer コマンド生成ツール

## 概要

`npa_generate_commands.py` は、CSVの各行を **`oci vn-monitoring path-analysis get-path-analysis-adhoc` コマンドに変換するだけ**のスクリプトです。

- Python標準ライブラリのみで動作し、OCI SDKは不要です。
- **PythonからOCIに接続しません**。分析の開始、完了待ち、結果取得、結果CSV出力は行いません。
- 生成される `.sh` にはCSV 1行につきOCI CLIコマンドが1つ入ります。内容を確認し、必要なコマンドを手動で実行してください。

## 構成

- `npa_generate_commands.py`: Pythonスクリプト
- `targets_sample.csv`: 入力CSV例
- `npa_commands_sample.sh`: 上記CSVから生成したコマンドの例（サンプルOCIDは実値に置換）

## 使い方

### 1. 入力CSVを用意する

ヘッダーは以下のとおりです。

```csv
source_ip,destination_ip,protocol,destination_port,source_type,source_subnet_id,destination_type,destination_subnet_id,source_port
192.168.10.10,10.0.1.10,TCP,22,ON_PREM,,OCI,,
192.168.10.10,10.0.1.10,TCP,443,ON_PREM,,OCI,,
10.0.1.10,192.168.10.20,TCP,3389,OCI,,ON_PREM,,
10.0.1.10,10.0.2.10,UDP,53,OCI,ocid1.subnet.oc1..REPLACE_SOURCE,OCI,ocid1.subnet.oc1..REPLACE_DESTINATION,
```

| 列 | 必須 | 内容 |
|---|---|---|
| `source_ip` | ○ | 送信元IPv4アドレス |
| `destination_ip` | ○ | 宛先IPv4アドレス |
| `protocol` | ○ | `TCP` または `UDP` |
| `destination_port` | ○ | 宛先ポート（1～65535） |
| `source_type` | 任意 | `OCI` / `ON_PREM`（空欄はOCI） |
| `destination_type` | 任意 | `OCI` / `ON_PREM`（空欄はOCI） |
| `source_subnet_id` | 任意 | OCI送信元サブネットのOCID。省略時はIPアドレスのみ指定 |
| `destination_subnet_id` | 任意 | OCI宛先サブネットのOCID。省略時はIPアドレスのみ指定 |
| `source_port` | 任意 | 送信元ポート（1～65535） |

※ オンプレミス側のサブネットOCIDは指定できません。IPアドレスに合うリソースが複数ある場合は、OCI側でサブネットOCIDを指定してください。

### 2. Bashコマンドを生成する

```bash
python npa_generate_commands.py \
  --input targets_sample.csv \
  --output npa_commands.sh \
  --compartment-id 'ocid1.compartment.oc1..実際のOCID' \
  --region ap-osaka-1
```

`--region` と `--profile` は省略できます。省略時は生成コマンド内に付与されず、CLI側の設定が使われます。

### 3. 生成コマンドを確認して手動で実行する

```bash
cat npa_commands.sh
```

表示されたOCI CLIコマンドから必要なものをコピーして実行できます。`bash npa_commands.sh` を実行すると **全件の分析が開始される**ため、手動確認が目的なら必要な行だけを実行してください。

## 注意事項

- `get-path-analysis-adhoc` は非同期の分析開始APIです。生成したCLIコマンドの応答は作業リクエストIDであり、到達可否そのものではありません。
- コマンド生成時に **双方向分析** (`--query-options '{"isBiDirectionalAnalysis":true}'`) を付与します。
- `npa_commands_sample.sh` のOCIDはダミーです。実際に使用する前に変更してください。
- Pythonの実行にOCI認証は不要ですが、**生成したCLIコマンドの実行にはOCI CLIの設定と必要な権限が必要です**。
- IPv4、TCP/UDPの範囲で利用する想定です。ローカル構文チェックは実施していますが、実OCI環境への接続テストは未実施です。

## 参考情報

- [OCI CLI: get-path-analysis-adhoc](https://docs.oracle.com/en-us/iaas/tools/oci-cli/latest/oci_cli_docs/cmdref/vn-monitoring/path-analysis/get-path-analysis-adhoc.html)
- [OCI ネットワーク・パス・アナライザ](https://docs.oracle.com/ja-jp/iaas/Content/Network/Concepts/path_analyzer.htm)

## ライセンス

本プロジェクトは [MIT License](LICENSE) のもとで公開しています。

