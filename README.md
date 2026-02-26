# Tự động hóa Hạ tầng Mạng Doanh nghiệp

## Tổng quan dự án
Dự án này mô phỏng hệ thống mạng doanh nghiệp đa chi nhánh có tính bảo mật và khả năng mở rộng cao, với trọng tâm là Tự động hóa mạng. Hạ tầng được xây dựng trên EVE-NG, sử dụng Firewall Fortinet và Switch Cisco để kết nối Site 1 tới Site 2 thông qua IPSec VPN. Các quy trình vận hành mạng, từ sao lưu cấu hình đến triển khai chính sách bảo mật, đều được tự động hóa hoàn toàn bằng Ansible.

## Kiến trúc và Sơ đồ mạng

*<img width="1176" height="827" alt="image" src="https://github.com/user-attachments/assets/1df275dd-25f5-4ee4-a6c1-0b7a711ca934" />*

Hệ thống mạng tuân theo kiến trúc Hub-and-Spoke qua môi trường Internet:
* Site 1 - Hà Nội: Sử dụng tường lửa FortiGate làm Gateway chính (Hub). Cơ sở này chứa Ansible Control Node (Ubuntu Linux) được đặt tại một mạng quản trị riêng biệt (VLAN 99).
* Site 2 - HCM: Sử dụng FortiGate làm Spoke.
* Kết nối: Đường hầm IPsec VPN Site-to-Site bảo mật nối giữa hai cơ sở, cho phép định tuyến và trao đổi dữ liệu nội bộ an toàn.

## Quy hoạch IP và VLAN
Mạng được phân chia thành các phòng ban cụ thể sử dụng VLAN, với dải IP được đồng bộ tương ứng giữa hai cơ sở để tối ưu hóa việc quản lý và định tuyến:

| Phân vùng | Site 1 (Hà Nội) | Site 2 (HCM) | Ghi chú |
| :--- | :--- | :--- | :--- |
| Management (VLAN 99/199) | 10.0.99.0/24 | 10.0.199.0/24 | Chứa Linux Ansible |
| Phòng Giáo viên (VLAN 110/210) | 10.0.110.0/24 | 10.0.210.0/24 | Đồng bộ đầu 10 |
| Phòng Lab (VLAN 120/220) | 10.0.120.0/24 | 10.0.220.0/24 | Đồng bộ đầu 20 |
| Phòng Học (VLAN 130/230) | 10.0.130.0/24 | 10.0.230.0/24 | Đồng bộ đầu 30 |

## Công nghệ sử dụng
* Bảo mật & Gateway: Fortinet FortiGate (FortiOS).
* Kết nối: IPsec VPN (Site-to-Site), Routing (Static/OSPF).
* Chuyển mạch: Cisco Switch Layer 2/Layer 3, VLAN Segmentation, 802.1Q Trunking.
* Tự động hóa (IaC): Ansible, Linux (Ubuntu Server), YAML.
* Môi trường Lab: EVE-NG.

## Tính năng Tự động hóa (Ansible Playbooks)
1. Cấu hình thiết bị ban đầu: Các Playbook `deploy_fg_...` và `deploy_sw_...` tự động thiết lập thông số cơ sở cho Firewall và Switch.
2. Triển khai VPN: Playbook `deploy_vpn_s2s.yml` thiết lập đường hầm IPsec Phase 1 & 2 giữa các site.
3. Quản trị chính sách tập trung: Tự động đẩy các rule tường lửa, chặn truy cập và lọc web (`deploy_policy_...`, `deploy_deny_...`, `deploy_webfilter_...`) đồng loạt xuống thiết bị.
4. Phục hồi sau sự cố (Disaster Recovery): Chạy tự động qua Cronjob (`backup_site1.yml`) để lấy cấu hình định kỳ lưu vào thư mục phân loại theo ngày tháng.

## Cấu trúc thư mục mã nguồn
```text
.
├── ansible.cfg
├── backup_cron.log
├── backups
│   ├── FG_S1
│   │   └── config_2026-02-23.conf
│   └── SW_S1
│       └── config_2026-02-23.cfg
├── backup_site1.yml
├── deploy_deny_policy_site1.yml
├── deploy_fg_site1.yml
├── deploy_fg_site2.yml
├── deploy_policy_site1.yml
├── deploy_policy_site2.yml
├── deploy_sw_site1.yml
├── deploy_vpn_s2s.yml
├── deploy_webfilter_site1.yml
├── group_vars
│   ├── all.yml
│   ├── site1_gateways.yml
│   ├── site1_switches.yml
│   ├── site2_gateways.yml
│   └── site2_switches.yml
├── hosts
└── host_vars
    ├── FG_S1.yml
    ├── FG_S2.yml
    ├── SW_S1.yml
    └── SW_S2.yml
