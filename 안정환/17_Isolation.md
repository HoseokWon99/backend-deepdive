### **이상 현상(Anomalies)과 격리 수준(Isolation Level)**

격리성을 완벽하게 지키지 못할 경우 발생하는 데이터 불일치 현상들입니다.

- **Dirty Read**: 커밋되지 않은 데이터를 읽음.
- **Non-Repeatable Read**: 반복해서 같은 데이터를 읽을 때 결과가 다름 (데이터 값이 수정됨).
- **Phantom Read**: 없던 데이터가 나타남 (데이터 행이 추가/삭제됨).
- **Lost Update**: 갱신 내용이 덮어쓰여 사라짐.
- **Dirty Write**: 커밋되지 않은 데이터를 덮어씀.
- **Read/Write Skew.. ETC**

### **격리 수준 (Isolation Level)**

| **Isolation Level**  | **Dirty Read** | **Non-Repeatable Read** | **Phantom Read** |
| -------------------- | -------------- | ----------------------- | ---------------- |
| **Read Uncommitted** | O              | O                       | O                |
| **Read Committed**   | X              | O                       | O                |
| **Repeatable Read**  | X              | X                       | O                |
| **Serializable**     | X              | X                       | X                |

---
