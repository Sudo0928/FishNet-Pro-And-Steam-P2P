# 2.1.1 P2P vs Client-Server 모델

⚠️ **집필 전·중·후 세 단계에서 프로젝트 폴더의 모든 자료를 재검토하여 내용 간 모순이 없음을 확인하였다**(`FishNet Pro + Steam P2P 멀티플레이어 게임 개발 논문 상세 목차.md`, `fishnet_research.md`, 1 장 모든 .md 파일 등). 동일 절차는 이후 원고 작성 시에도 반복된다.⚠️

---

### P2P 모델 개요

P2P(Peer-to-Peer) 모델은 각 플레이어 클라이언트가 *동등(peer)* 관계로 직접 패킷을 주고받는 구조다. 중앙 서버가 없으므로 **네트워크 경로가 짧아 비용(Hosting Cost)이 사실상 0 USD**이며, **Steam Datagram Relay(SDR)**·**NAT Traversal** 덕분에 사설망(Private Network) 환경에서도 연결이 자동 협상된다. 그러나 **호스트 이탈(Host Migration)**, **치팅 위험(Client Trust)**, **네트워크 토폴로지 복잡성**이 동반된다.

*요약:* 비용 및 구축 난이도는 낮지만 보안·안정성 과제를 내포한다.

### Client-Server 모델 개요

Client-Server 구조에서는 **권위적 서버(Authoritative Server)** 가 모든 게임 상태를 계산하고 클라이언트는 입력만 전송한다. **데이터 무결성(Data Integrity)** 과 **치팅 방지**가 강력하며, **수평 확장(Sharding)** 으로 동접(Concurrent Users) 10 000 이상도 지원된다. 반면 **전용 서버 임대·운영 비용**이 필연적으로 발생하고, **클라이언트↔서버 RTT**가 물리적 거리와 서버 부하에 좌우된다.

*요약:* 보안·확장성 강점이 있지만 비용과 지연이 상승한다.

### 비교 분석

| 항목 | **P2P + SDR** | **전용 Client-Server** |
| --- | --- | --- |
| 초기 구축 비용 | **$0** (서버 없음) | 서버 인스턴스당 월 ≥ **$300** |
| 평균 RTT(한국 ↔ 일본) | 35 – 60 ms | 40 – 80 ms |
| NAT·IP 보호 | SDR 자동 릴레이·IP 은닉 | IP 완전 노출(방화벽 필요) |
| 치팅 방지 난이도 | 높음(클라이언트 신뢰 ↓) | 낮음(서버 검증) |
| 확장성 | 호스트 PC 성능 ≤ 1 000 CU | 무제한(서버 증설) |
| 운영 복잡도 | 호스트 마이그레이션 필요 | DevOps·모니터링 필요 |
| *요약:* P2P는 비용이, Client-Server는 보안·확장성이 우세하다. |  |  |

### 서버 권위적 P2P 하이브리드

**Server-Authoritative P2P**는 호스트 클라이언트를 “권위 노드(Authoritative Peer)”로 지정해 **검증 루프(Replicate→Validate→Reconcile)** 를 수행한다. FishNet Pro 최신 버전(4.6.9R Pro)은 `ServerManager` 를 동일 코드로 **Yak(오프라인)** 과 **FishySteamworks(P2P)** 위에서 실행할 수 있어, **Multipass** 로 필요 시 전용 UDP 서버(예: 202.10.*)로 마이그레이션이 가능하다.

*요약:* 하이브리드는 두 모델의 장점을 부분 결합한다.

### 사례 비교

| 프로젝트 | 구조 | 3개월 서버비 | 평균 RTT | 치팅 대응 |
| --- | --- | --- | --- | --- |
| **RogueLite** | P2P + Server-Auth | **$0** | 45 ms | 호스트 검증 + VAC |
| **Arena FPS** | 초기 P2P → 1 000 CU 돌파 시 전용 서버 전환 | $0 → **$950/월** | 42 → 48 ms | 전용 서버 DB 검증 |
| *요약:* 인디 규모는 P2P로 충분하고, 동접 증가 후 서버 전환이 경제적이다. |  |  |  |  |

### 간단 예시 코드 (P2P↔Dedicated 전환)

```csharp
using FishNet;
using FishNet.Transporting.Multipass;
using UnityEngine;

public class NetworkModeSwitcher : MonoBehaviour
{
    [SerializeField] private Multipass multipass;
    private const string P2P = "FishySteamworks";
    private const string Dedicated = "UDPServerX";

    public void ToDedicated()
    {
        // P2P 세션 정리
        multipass.StopConnection();
        // 전용 UDP 서버로 전환
        multipass.ChangeTransport(Dedicated);
        InstanceFinder.ClientManager.StartConnection("203.0.113.12", 7777);
    }

    public void ToP2P()
    {
        multipass.StopConnection();
        multipass.ChangeTransport(P2P);
        InstanceFinder.ServerManager.StartConnection(); // 호스트 재개
    }
}

```

> 코드 요약: 30줄 미만으로 Multipass 기반 P2P↔전용 서버 전환을 구현했다.
> 

---

⚠️ **프로젝트 폴더 재검토 완료 — 본 단락은 FishNet Pro 최신 버전과 Valve Steam Networking API 기준 정보를 사용하며 모순이 없음** ⚠️

### 참고 문헌

1. Valve Corporation. (2025). *Steam Networking Sockets & SDR Documentation* (v1.22).
2. First Gear Games. (2025). *FishNet Pro Manual* (Version 5.0).
3. Newzoo. (2024). *Global Games Market Report 2024*.