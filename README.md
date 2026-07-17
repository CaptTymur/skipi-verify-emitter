# skipi-verify-emitter

Внешний эмиттер merge-гейта `skipi-broker` (Вариант A, SKI-INC-2026-07-17).

**Функция:** единственный доверенный источник commit-status `verify` на Broker.
Cron-sweeper перечисляет открытые PR skipi-broker, для каждого берёт точный
`head.sha`, прогоняет НЕИЗМЕНЯЕМЫЙ checker `skipi-verify@<immutable SHA>`
(байт-пины; пин живёт в workflow этого репо, НЕ на стороне Broker) и постит
статус через GitHub App `skipi-verify-emitter`. Код проверяемых PR НЕ
исполняется. Нет прогона → нет статуса → merge blocked (fail-closed).

**Ключ App:** ТОЛЬКО environment secret `VERIFY_EMITTER_APP_KEY` в окружении
`emitter` (deployment branch policy: только `main`, без required reviewer).
Repository secrets ЗАПРЕЩЕНЫ (утекают в PR-workflow — доказано POC
`emitter-secret-isolation-poc`). Ключ не должен проходить через сессии агентов.

**Почему public:** environment protection rules на free-плане работают только
в public-репо; секретов в коде нет by design.

**Недоверие промежуточному зелёному (решение Тимура 17.07):** зелёному статусу
от эмиттера НЕ доверять, пока не пройдена вся цепочка — EXEC-PAT с
доказательством push→403/404, protection Broker к стандарту, три
BLOCKED-контроля (NC1–NC3) — и независимая приёмка супервайзора.
