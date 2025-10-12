
# OCI ARM 实例自动管理脚本

## 功能简介
- 每小时自动检查租户是否存在实例
- 若无实例则自动创建新的 Ubuntu ARM 实例（4C24G）
- 操作系统：Canonical Ubuntu 22.04 Minimal aarch64
- 实例类型：VM.Standard.A1.Flex（Oracle Cloud 始终免费）

---

## 工作流程说明

### 触发条件
- **自动触发**：每小时整点执行一次
- **手动触发**：可通过 GitHub Actions 界面手动启动

### 执行逻辑
1. 检查租户中是否存在任何实例（所有状态都会检查）
2. 如果发现任何实例存在：
	- 立即终止工作流
	- 不进行任何实例操作
3. 如果确认没有任何实例：
	- 创建新的 Ubuntu ARM 实例
	- 配置 4 OCPU 和 24GB 内存
	- 分配公网 IP
	- 注入 SSH 密钥

### 使用限制
- 每个租户同一时间只允许存在一个实例
- 创建新实例前必须确保无其他实例存在
- 实例规格严格遵守免费套餐限制

---

## 所需 GitHub Secrets 说明

| 名称              | 说明                                                         | 示例/获取方式 |
|-------------------|--------------------------------------------------------------|--------------|
| **OCI_USER**      | Oracle Cloud 用户唯一标识符<br>控制台 → 用户设置 → 用户OCID  | `ocid1.user.oc1..aaaaaaaaxxxxxxxx` |
| **OCI_TENANCY**   | Oracle Cloud 租户唯一标识符<br>控制台 → 租户信息 → 租户OCID | `ocid1.tenancy.oc1..aaaaaaaaxxxxxxxx` |
| **OCI_FINGERPRINT** | API密钥指纹<br>控制台 → 个人资料 → API密钥                  | `aa:bb:cc:dd:...` |
| **OCI_PRIVATE_KEY** | API认证私钥（PEM格式，未加密）<br>使用 openssl 生成         | `-----BEGIN RSA PRIVATE KEY----- ...` |
| **OCI_REGION**    | 区域标识符<br>推荐：us-ashburn-1, ap-seoul-1, eu-frankfurt-1 | `us-ashburn-1` |
| **SSH_PUBLIC_KEY** | 用于 SSH 登录的公钥<br>使用 ssh-keygen 生成                 | `ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB...` |
| **OCI_COMPARTMENT** | 要操作的 Compartment OCID（用于 --compartment-id 参数）<br>控制台 → Identity → Compartments | `ocid1.compartment.oc1..aaaa...` |

> 获取方式可参考 Oracle Cloud 控制台相关页面。

---

## 快速链接（Oracle Cloud 控制台）

以下链接可直接打开 Oracle Cloud 控制台相关页面，用于查找或生成上表提到的参数：

- OCI 用户（User OCID）：https://cloud.oracle.com/identity/domains/my-profile
- 租户（Tenancy OCID）：https://cloud.oracle.com/tenancy
- API 密钥与指纹（Fingerprint / Private Key）：https://cloud.oracle.com/identity/domains/my-profile/auth-tokens
- 区域列表（Regions）：https://cloud.oracle.com/regions
- 区域(Compartment OCID):https://cloud.oracle.com/identity/domains/

提示：控制台页面可能需要先登录 Oracle Cloud 帐户。

## 安全注意事项

- 所有 Secrets 都应严格保密，不要在代码中硬编码
- 定期轮换 API 密钥和 SSH 密钥（建议每 3-6 个月）
- 为 OCI 用户分配最小必要权限
- 启用 GitHub 仓库的分支保护和访问控制

---

如需详细操作步骤或遇到问题，请参考 Oracle Cloud 官方文档或联系仓库维护者。