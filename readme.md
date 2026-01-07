# komari-agent

```shell
docker run -d \
  --name komari-agent \
  --restart unless-stopped \
  ghcr.io/komari-monitor/komari-agent:latest \
  --endpoint www \
  --token abc
```

```shell
docker run -d \
  --name komari-agent \
  --restart unless-stopped \
  -e AGENT_TOKEN=abc \
  -e AGENT_ENDPOINT=www \
  ghcr.io/arlettebrook/komari-agent:latest
```


---

支持使用环境变量 / JSON配置文件来传入 agent 参数

详见 `cmd/flags/flags.go` 及 `cmd/root.go`



```go
package flags_pkg

type Config struct {
	AutoDiscoveryKey     string  `json:"auto_discovery_key" env:"AGENT_AUTO_DISCOVERY_KEY"`           // 自动发现密钥
	DisableAutoUpdate    bool    `json:"disable_auto_update" env:"AGENT_DISABLE_AUTO_UPDATE"`         // 禁用自动更新
	DisableWebSsh        bool    `json:"disable_web_ssh" env:"AGENT_DISABLE_WEB_SSH"`                 // 禁用远程控制（web ssh 和 rce）
	MemoryModeAvailable  bool    `json:"memory_mode_available" env:"AGENT_MEMORY_MODE_AVAILABLE"`     // [deprecated] 已弃用，请使用 MemoryIncludeCache
	Token                string  `json:"token" env:"AGENT_TOKEN"`                                     // Token
	Endpoint             string  `json:"endpoint" env:"AGENT_ENDPOINT"`                               // 面板地址
	Interval             float64 `json:"interval" env:"AGENT_INTERVAL"`                               // 数据采集间隔，单位秒
	IgnoreUnsafeCert     bool    `json:"ignore_unsafe_cert" env:"AGENT_IGNORE_UNSAFE_CERT"`           // 忽略不安全的证书
	MaxRetries           int     `json:"max_retries" env:"AGENT_MAX_RETRIES"`                         // 最大重试次数
	ReconnectInterval    int     `json:"reconnect_interval" env:"AGENT_RECONNECT_INTERVAL"`           // 重连间隔，单位秒
	InfoReportInterval   int     `json:"info_report_interval" env:"AGENT_INFO_REPORT_INTERVAL"`       // 基础信息上报间隔，单位分钟
	IncludeNics          string  `json:"include_nics" env:"AGENT_INCLUDE_NICS"`                       // 仅统计网卡，逗号分隔的网卡名称列表，支持通配符
	ExcludeNics          string  `json:"exclude_nics" env:"AGENT_EXCLUDE_NICS"`                       // 统计时排除的网卡，逗号分隔的网卡名称列表，支持通配符
	IncludeMountpoints   string  `json:"include_mountpoints" env:"AGENT_INCLUDE_MOUNTPOINTS"`         // 磁盘统计的包含挂载点列表，使用分号分隔
	MonthRotate          int     `json:"month_rotate" env:"AGENT_MONTH_ROTATE"`                       // 流量统计的月份重置日期（0表示禁用）
	CFAccessClientID     string  `json:"cf_access_client_id" env:"AGENT_CF_ACCESS_CLIENT_ID"`         // Cloudflare Access Client ID
	CFAccessClientSecret string  `json:"cf_access_client_secret" env:"AGENT_CF_ACCESS_CLIENT_SECRET"` // Cloudflare Access Client Secret
	MemoryIncludeCache   bool    `json:"memory_include_cache" env:"AGENT_MEMORY_INCLUDE_CACHE"`       // 包括缓存/缓冲区的内存使用情况
	MemoryReportRawUsed  bool    `json:"memory_report_raw_used" env:"AGENT_MEMORY_REPORT_RAW_USED"`   // 使用原始内存使用情况报告
	CustomDNS            string  `json:"custom_dns" env:"AGENT_CUSTOM_DNS"`                           // 使用的自定义DNS服务器
	EnableGPU            bool    `json:"enable_gpu" env:"AGENT_ENABLE_GPU"`                           // 启用详细GPU监控
	ShowWarning          bool    `json:"show_warning" env:"AGENT_SHOW_WARNING"`                       // Windows 上显示安全警告，作为子进程运行一次
	CustomIpv4           string  `json:"custom_ipv4" env:"AGENT_CUSTOM_IPV4"`                         // 自定义 IPv4 地址
	CustomIpv6           string  `json:"custom_ipv6" env:"AGENT_CUSTOM_IPV6"`                         // 自定义 IPv6 地址
	GetIpAddrFromNic     bool    `json:"get_ip_addr_from_nic" env:"AGENT_GET_IP_ADDR_FROM_NIC"`       // 从网卡获取IP地址
	ConfigFile           string  `json:"config_file" env:"AGENT_CONFIG_FILE"`                         // JSON配置文件路径

}

var GlobalConfig = &Config{}
```

```go
func init() {
        RootCmd.PersistentFlags().StringVarP(&flags.Token, "token", "t", "", "API token")
        //RootCmd.MarkPersistentFlagRequired("token")
        RootCmd.PersistentFlags().StringVarP(&flags.Endpoint, "endpoint", "e", "", "API endpoint")
        //RootCmd.MarkPersistentFlagRequired("endpoint")
        RootCmd.PersistentFlags().StringVar(&flags.AutoDiscoveryKey, "auto-discovery", "", "Auto discovery key for the agent")
        RootCmd.PersistentFlags().BoolVar(&flags.DisableAutoUpdate, "disable-auto-update", false, "Disable automatic updates")
        RootCmd.PersistentFlags().BoolVar(&flags.DisableWebSsh, "disable-web-ssh", false, "Disable remote control(web ssh and rce)")
        //RootCmd.PersistentFlags().BoolVar(&flags.MemoryModeAvailable, "memory-mode-available", false, "[deprecated]Report memory as available instead of used.")
        RootCmd.PersistentFlags().Float64VarP(&flags.Interval, "interval", "i", 1.0, "Interval in seconds")
        RootCmd.PersistentFlags().BoolVarP(&flags.IgnoreUnsafeCert, "ignore-unsafe-cert", "u", false, "Ignore unsafe certificate errors")
        RootCmd.PersistentFlags().IntVarP(&flags.MaxRetries, "max-retries", "r", 3, "Maximum number of retries")
        RootCmd.PersistentFlags().IntVarP(&flags.ReconnectInterval, "reconnect-interval", "c", 5, "Reconnect interval in seconds")
        RootCmd.PersistentFlags().IntVar(&flags.InfoReportInterval, "info-report-interval", 5, "Interval in minutes for reporting basic info")
        RootCmd.PersistentFlags().StringVar(&flags.IncludeNics, "include-nics", "", "Comma-separated list of network interfaces to include")
        RootCmd.PersistentFlags().StringVar(&flags.ExcludeNics, "exclude-nics", "", "Comma-separated list of network interfaces to exclude")
        RootCmd.PersistentFlags().StringVar(&flags.IncludeMountpoints, "include-mountpoint", "", "Semicolon-separated list of mount points to include for disk statistics")
        RootCmd.PersistentFlags().IntVar(&flags.MonthRotate, "month-rotate", 0, "Month reset for network statistics (0 to disable)")
        RootCmd.PersistentFlags().StringVar(&flags.CFAccessClientID, "cf-access-client-id", "", "Cloudflare Access Client ID")
        RootCmd.PersistentFlags().StringVar(&flags.CFAccessClientSecret, "cf-access-client-secret", "", "Cloudflare Access Client Secret")
        RootCmd.PersistentFlags().BoolVar(&flags.MemoryIncludeCache, "memory-include-cache", false, "Include cache/buffer in memory usage")
        RootCmd.PersistentFlags().BoolVar(&flags.MemoryReportRawUsed, "memory-exclude-bcf", false, "Use \"raminfo.Used = v.Total - v.Free - v.Buffers - v.Cached\" calculation for memory usage")
        RootCmd.PersistentFlags().StringVar(&flags.CustomDNS, "custom-dns", "", "Custom DNS server to use (e.g. 8.8.8.8, 114.114.114.114). By default, the program uses the system DNS resolver.")
        RootCmd.PersistentFlags().BoolVar(&flags.EnableGPU, "gpu", false, "Enable detailed GPU monitoring (usage, memory, multi-GPU support)")
        RootCmd.PersistentFlags().BoolVar(&flags.ShowWarning, "show-warning", false, "Show security warning on Windows, run once as a subprocess")
        RootCmd.PersistentFlags().StringVar(&flags.CustomIpv4, "custom-ipv4", "", "Custom IPv4 address to use")
        RootCmd.PersistentFlags().StringVar(&flags.CustomIpv6, "custom-ipv6", "", "Custom IPv6 address to use")
        RootCmd.PersistentFlags().BoolVar(&flags.GetIpAddrFromNic, "get-ip-addr-from-nic", false, "Get IP address from network interface")
        RootCmd.PersistentFlags().StringVar(&flags.ConfigFile, "config", "", "Path to the configuration file")
        RootCmd.PersistentFlags().ParseErrorsWhitelist.UnknownFlags = true
}
```

---

