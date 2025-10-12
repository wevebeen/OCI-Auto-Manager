# ==============================================================================
# Oracle Cloud ARM实例自动管理脚本
# 功能：每小时检查租户中是否存在实例，如无实例则创建新的Ubuntu ARM实例（4C24G）
# 操作系统：Canonical Ubuntu 22.04 Minimal aarch64
# 实例类型：VM.Standard.A1.Flex（符合Oracle Cloud始终免费条件）
# ==============================================================================

# 工作流程说明
# ==============================================================================
# 1. 触发条件：
#    - 自动触发：每小时整点执行一次
#    - 手动触发：可通过GitHub Actions界面手动启动
#
# 2. 执行逻辑：
#    a) 检查租户中是否存在任何实例（所有状态都会检查）
#    b) 如果发现任何实例存在：
#       - 立即终止工作流
#       - 不进行任何实例操作
#    c) 如果确认没有任何实例：
#       - 创建新的Ubuntu ARM实例
#       - 配置4 OCPU和24GB内存
#       - 分配公网IP
#       - 注入SSH密钥
#
# 3. 使用限制：
#    - 每个租户同一时间只允许存在一个实例
#    - 创建新实例前必须确保无其他实例存在
#    - 实例规格严格遵守免费套餐限制
# ==============================================================================

# ==============================================================================
# 所需GitHub Secrets详细说明
# ==============================================================================
#
# 1. OCI_USER (必需)
#    - 功能：Oracle Cloud用户唯一标识符(获取)[https://cloud.oracle.com/identity/domains/my-profile]
#    - 获取方式：Oracle Cloud控制台 → 右上角头像 → 用户设置 → 用户OCID
#    - 示例格式：ocid1.user.oc1..aaaaaaaaxxxxxxxx
#
# 2. OCI_TENANCY (必需)
#    - 功能：Oracle Cloud租户（账户）唯一标识符(获取)[https://cloud.oracle.com/tenancy]
#    - 获取方式：Oracle Cloud控制台 → 租户信息 → 租户OCID
#    - 示例格式：ocid1.tenancy.oc1..aaaaaaaaxxxxxxxx
#
# 3. OCI_FINGERPRINT (必需)
#    - 功能：API密钥的指纹，用于验证密钥完整性(获取)[https://cloud.oracle.com/identity/domains/my-profile/auth-tokens]
#    - 获取方式：API 密钥添加，选择下载的公钥和私钥，然后选择公钥文件在添加，3-4步骤在同一个地方
#    - 示例格式：aa:bb:cc:dd:ee:ff:11:22:33:44:55:66:77:88:99:00

# 4. OCI_PRIVATE_KEY (必需)
#    - 功能：API认证的私钥，用于签署API请求[https://cloud.oracle.com/identity/domains/my-profile/auth-tokens]
#    - 生成方式：使用openssl生成RSA密钥对
#    - 格式要求：完整的PEM格式，包含BEGIN和END标记
#    - 安全要求：必须是未加密的私钥
#    - 示例格式：
#        -----BEGIN RSA PRIVATE KEY-----
#        MIIEpAIBAAKCAQEA...
#        ...
#        -----END RSA PRIVATE KEY-----
#
# 5. OCI_REGION (必需)
#    - 功能：指定操作的Oracle Cloud区域(获取)[https://cloud.oracle.com/regions]
#    - 推荐值：us-ashburn-1 (北美), ap-seoul-1 (亚洲), eu-frankfurt-1 (欧洲)
#    - 示例格式：us-ashburn-1
#
# 6. SSH_PUBLIC_KEY (必需)
#    - 功能：注入到创建的实例中，用于SSH密钥认证登录
#    - 生成方式：ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
#    - 格式要求：标准SSH公钥格式
#    - 示例格式：ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB...
#
# 安全注意事项：
# - 所有Secrets都应严格保密，不要在代码中硬编码
# - 定期轮换API密钥和SSH密钥（建议每3-6个月）
# - 为OCI用户分配最小必要权限
# - 启用GitHub仓库的分支保护和访问控制
# ==============================================================================