<h1>
  Kubernetes <img src="./assets/kubernetes.svg" width="40" height="40" />
</h1>

<h2>Найпопулярніші запитання та відповіді на співбесіді з Kubernetes</h2>

<details>
<summary>1. Що таке Kubernetes?</summary>

#### Kubernetes

Kubernetes (K8s) — це open-source платформа оркестрації контейнерів для
декларативного розгортання, масштабування та керування застосунками.

Kubernetes працює за desired state моделлю: ти описуєш стан у YAML, а control
plane постійно приводить фактичний стан до цільового.

Ключові обʼєкти в роботі:

- `Pod` — мінімальна одиниця запуску контейнерів;
- `Deployment`/`StatefulSet`/`DaemonSet` — керування життєвим циклом workload;
- `Service`, `Ingress`, `Gateway API` — мережевий доступ і маршрутизація;
- `ConfigMap`, `Secret` — конфігурація і секрети.

**Коротко:**

- Kubernetes автоматизує запуск і керування контейнеризованими застосунками.
- Працює декларативно через desired state та controllers.
- Дає стандартний API для compute, networking, storage і security.

</details>

<details>
<summary>2. Які проблеми вирішує Kubernetes?</summary>

#### Kubernetes

Kubernetes прибирає ручні операції навколо контейнерів і дає стабільну
production-експлуатацію.

Практичні проблеми, які він закриває:

- orchestration: планування Pod на вузли, self-healing, автоперезапуск;
- scaling: горизонтальне масштабування через `HPA`, кластерне через autoscaler;
- delivery: rolling update, rollback, zero-downtime деплой;
- service networking: discovery, балансування, політики мережі;
- config/security: централізовані `ConfigMap`/`Secret`, `RBAC`, pod security.

**Коротко:**

- Kubernetes зменшує операційний overhead у керуванні контейнерами.
- Дає вбудовані механізми відмовостійкості, масштабування і оновлень.
- Уніфікує delivery-процес між середовищами.

</details>

<details>
<summary>3. Що таке кластер Kubernetes?</summary>

#### Kubernetes

Кластер Kubernetes — це набір вузлів, обʼєднаних одним control plane і спільним
API.

Склад кластера:

- control plane: `kube-apiserver`, `etcd`, `scheduler`, `controller-manager`;
- worker nodes: `kubelet`, container runtime (containerd/CRI-O), `kube-proxy` або eBPF data plane;
- CNI-плагін для pod-to-pod мережі;
- CSI-драйвери для storage інтеграції.

Кластер може бути single-zone або multi-zone залежно від вимог HA.

**Коротко:**

- Кластер = control plane + worker nodes.
- Усі операції проходять через Kubernetes API.
- HA, network і storage можливості залежать від конфігурації кластера.

</details>

<details>
<summary>4. Що таке Node?</summary>

#### Kubernetes

Node — це обчислювальний вузол кластера (VM або bare metal), де запускаються
Pod.

На Node працюють:

- `kubelet` — агент, що виконує PodSpec і репортує стан;
- container runtime через CRI;
- компонент мережі (`kube-proxy` або CNI/eBPF datapath).

Node має ресурси (`CPU`, `RAM`, ephemeral storage), які scheduler враховує через
`requests/limits`, affinity, taints/tolerations.

**Коротко:**

- Node надає ресурси для запуску Pod.
- `kubelet` синхронізує workload із control plane.
- Планування Pod на Node керується політиками scheduling.

</details>

<details>
<summary>5. Що таке Pod?</summary>

#### Kubernetes

Pod — найменша deploy-одиниця в Kubernetes: один або кілька контейнерів із
спільним network namespace та томами.

Важливі властивості:

- один Pod IP для всіх контейнерів у Pod;
- локальний обмін між контейнерами через `localhost`;
- Pod ефемерний: при пересозданні змінюється ідентичність.

У production Pod зазвичай керується контролером (`Deployment`, `StatefulSet`),
а не створюється вручну.

**Коротко:**

- Pod інкапсулює контейнер(и), мережу та томи як єдину одиницю.
- Контейнери в Pod спільно використовують IP і можуть ділити storage.
- Для масштабування/оновлень Pod використовують workload controllers.

</details>

<details>
<summary>6. Що таке Pod з кількома контейнерами?</summary>

#### Kubernetes

Multi-container Pod — Pod, у якому кілька контейнерів працюють разом як один
функціональний юніт.

Коли це доречно:

- основний контейнер + sidecar для логування, проксі, auth, telemetry;
- тісно повʼязані процеси з однаковим життєвим циклом;
- потреба в обміні через `localhost` або спільний volume.

Коли не доречно:

- незалежні сервіси з різним lifecycle краще розділяти на окремі Pod.

**Коротко:**

- Multi-container Pod використовується для тісно звʼязаних процесів.
- Контейнери ділять мережу й томи, але мають окремі образи та процеси.
- Незалежні компоненти краще виносити в різні Pod.

</details>

<details>
<summary>7. Що таке sidecar-контейнер?</summary>

#### Kubernetes

Sidecar — допоміжний контейнер у тому ж Pod, що й основний застосунок, який
додає платформену функцію без зміни коду застосунку.

Типові сценарії:

- логування/форвардинг логів;
- service mesh proxy;
- cert/secret reload;
- локальний кеш або адаптер протоколів.

Sidecar має спільний життєвий контекст із Pod, тому його доступність впливає на
поведінку всього Pod.

**Коротко:**

- Sidecar розширює можливості основного контейнера в межах одного Pod.
- Найчастіше використовується для мережі, безпеки та observability.
- Це стандартний патерн для platform-level функцій.

</details>

<details>
<summary>8. Що таке native sidecar containers?</summary>

#### Kubernetes

Native sidecar containers — це sidecar-контейнери, оголошені через
`initContainers` з політикою перезапуску sidecar, щоб вони жили протягом усього
життєвого циклу Pod і запускались у передбачуваному порядку.

Практичний ефект:

- контрольований startup-order (спочатку sidecar, далі app-контейнери);
- стабільніша поведінка Pod при рестартах sidecar;
- чистіша модель для проксі/агентів, яким треба стартувати раніше застосунку.

Це зменшує кількість workaround-патернів для класичних sidecar у складних Pod.

**Коротко:**

- Native sidecar дає нативний lifecycle для допоміжних контейнерів.
- Покращує контроль порядку старту і перезапусків у Pod.
- Корисно для mesh proxy, security/telemetry агентів та bootstrap-залежностей.

</details>

<details>
<summary>9. Що таке Namespace?</summary>

#### Kubernetes

Namespace — логічна ізоляція ресурсів усередині одного кластера Kubernetes.

Що дає Namespace у production:

- розділення середовищ і команд (`dev`, `staging`, `prod`, team-per-namespace);
- scoped policy: `RBAC`, `ResourceQuota`, `LimitRange`, `NetworkPolicy`;
- зручніший операційний контур для deployment і доступів.

Приклад роботи:

```bash
kubectl get pods -n payments
kubectl apply -f deployment.yaml -n payments
kubectl auth can-i get secrets -n payments --as system:serviceaccount:payments:api
```

**Коротко:**

- Namespace ізолює ресурси і політики в межах одного кластера.
- Дає базовий контур для доступів, квот і мережевих правил.
- Є ключовою одиницею організації multi-team середовища.

</details>

<details>
<summary>10. Як Kubernetes підтримує multi-tenancy?</summary>

#### Kubernetes

Kubernetes підтримує multi-tenancy комбінацією логічної ізоляції, контролю
доступу та ресурсних обмежень.

Базовий production-набір:

- `Namespace` як межа tenant;
- `RBAC` для least-privilege доступу;
- `ResourceQuota` і `LimitRange` для fair usage ресурсів;
- `NetworkPolicy` для сегментації трафіку;
- `Pod Security Admission` + Pod Security Standards для базового hardening;
- окремі service accounts, secrets, CI/CD policy на tenant.

Для жорсткої ізоляції використовують окремі кластери або віртуальні кластери.

**Коротко:**

- Multi-tenancy в Kubernetes реалізується політиками, а не одним прапорцем.
- Мінімальний контур: Namespace + RBAC + Quotas + NetworkPolicy + Pod Security.
- Рівень ізоляції обирають між shared-cluster і dedicated-cluster моделями.

</details>

<details>
<summary>11. Опишіть архітектуру Kubernetes.</summary>

#### Kubernetes

Архітектура Kubernetes складається з control plane та worker nodes.

Control plane приймає декларації стану через API і керує кластером через
контролери. Worker nodes виконують Pod і звітують про стан.

Типовий потік:

1. Маніфест застосовується через `kubectl apply -f ...`.
2. `kube-apiserver` валідує запит і зберігає стан в `etcd`.
3. Контролери створюють/оновлюють потрібні обʼєкти.
4. `kube-scheduler` привʼязує Pod до Node.
5. `kubelet` на Node запускає контейнери через CRI runtime.
6. Мережа/сервіс-доступ надаються через CNI + Service/Gateway/Ingress.

**Коротко:**

- Kubernetes працює як control-loop система desired state.
- Control plane приймає рішення, nodes виконують workload.
- API + etcd + controllers + scheduler + kubelet формують основний цикл.

</details>

<details>
<summary>12. Що таке control plane?</summary>

#### Kubernetes

Control plane — набір керуючих компонентів кластера, які зберігають стан,
планують workload і підтримують узгодженість ресурсів.

Склад control plane:

- `kube-apiserver` — єдина точка входу в API;
- `etcd` — key-value сховище стану кластера;
- `kube-scheduler` — вибір Node для Pod;
- `kube-controller-manager` — control loops для ресурсів;
- `cloud-controller-manager` (у хмарі) — інтеграція з cloud API.

У HA-конфігурації control plane запускають у кількох репліках.

**Коротко:**

- Control plane керує всім життєвим циклом ресурсів Kubernetes.
- Критичні компоненти: API server, etcd, scheduler, controller manager.
- Від доступності control plane залежить керованість кластера.

</details>

<details>
<summary>13. Що таке kube-apiserver?</summary>

#### Kubernetes

`kube-apiserver` — центральний API-сервер Kubernetes, через який проходять усі
операції читання/запису ресурсів.

Функції `kube-apiserver`:

- автентифікація, авторизація (`RBAC`), admission;
- валідація/мутація обʼєктів;
- збереження стану в `etcd`;
- надання watch-стрімів для controllers/operators.

Практичні команди взаємодії:

```bash
kubectl get --raw /healthz
kubectl api-resources
kubectl explain deployment.spec.template.spec
```

**Коротко:**

- `kube-apiserver` є єдиною офіційною точкою доступу до кластера.
- Через нього проходять безпека, валідація та persistence стану.
- Усі controllers і клієнти працюють через Kubernetes API.

</details>

<details>
<summary>14. Що таке etcd і чому він критичний?</summary>

#### Kubernetes

`etcd` — розподілене консистентне key-value сховище, у якому Kubernetes тримає
весь кластерний стан (обʼєкти API, конфігурацію, метадані).

Чому критичний:

- втрата `etcd` = втрата джерела істини про кластер;
- продуктивність `etcd` впливає на latency control plane;
- консистентність `etcd` забезпечує коректну роботу controllers.

Практика production:

- регулярні snapshot backup;
- шифрування секретів at-rest;
- low-latency диски та моніторинг затримок/розміру БД.

**Коротко:**

- `etcd` зберігає весь desired/actual state Kubernetes.
- Це найкритичніший компонент control plane з точки зору даних.
- Backup/restore `etcd` має бути перевіреним операційним процесом.

</details>

<details>
<summary>15. Що таке kube-scheduler?</summary>

#### Kubernetes

`kube-scheduler` призначає Pod на конкретний Node, якщо Pod ще не має привʼязки.

При виборі Node scheduler враховує:

- `requests/limits` і доступні ресурси;
- `nodeSelector`, node affinity/anti-affinity;
- taints/tolerations;
- topology spread constraints;
- policy-пріоритети та scoring rules.

Scheduler не запускає контейнери самостійно, а лише робить binding Pod->Node.

**Коротко:**

- `kube-scheduler` відповідає за placement Pod у кластері.
- Рішення базується на ресурсах і scheduling-політиках.
- Після binding запуск контейнерів виконує `kubelet` на вибраному Node.

</details>

<details>
<summary>16. Що таке kube-controller-manager?</summary>

#### Kubernetes

`kube-controller-manager` запускає набір control loops, що постійно звіряють
desired state з фактичним станом та коригують розбіжності.

Приклади контролерів:

- Deployment/ReplicaSet controller;
- Node controller;
- Job/CronJob controller;
- ServiceAccount, Namespace та інші базові контролери.

Кожен контролер працює за схемою `watch -> compare -> reconcile`.

**Коротко:**

- `kube-controller-manager` реалізує reconciliation у Kubernetes.
- Контролери автоматично повертають систему до описаного стану.
- Це основа self-healing і декларативної моделі.

</details>

<details>
<summary>17. Що таке kubelet?</summary>

#### Kubernetes

`kubelet` — агент на кожному Node, який забезпечує виконання PodSpec на цьому
вузлі.

Що робить `kubelet`:

- отримує Pod-опис із API server;
- запускає/зупиняє контейнери через CRI runtime;
- монтує томи, налаштовує probes;
- репортує статус Pod/Node у control plane.

`kubelet` не приймає глобальні рішення планування, це зона scheduler.

**Коротко:**

- `kubelet` керує життєвим циклом Pod на конкретному Node.
- Взаємодіє з runtime через CRI і зі storage/network підсистемами.
- Саме `kubelet` забезпечує виконання того, що призначив control plane.

</details>

<details>
<summary>18. Що таке kube-proxy?</summary>

#### Kubernetes

`kube-proxy` — компонент Node-рівня, який реалізує мережеву модель Service
(ClusterIP/NodePort/частково LoadBalancer backend routing) через правила
маршрутизації.

Залежно від режиму використовує `iptables`, `ipvs` або eBPF-реалізацію
мережевого стеку (через CNI-рішення).

Основна задача:

- направити трафік із Service VIP на актуальні Pod endpoints.

**Коротко:**

- `kube-proxy` забезпечує сервісну маршрутизацію всередині кластера.
- Працює на кожному Node і оновлює мережеві правила за змінами endpoints.
- Є критичним для стабільної Service connectivity.

</details>

<details>
<summary>19. Що таке Container Runtime Interface (CRI)?</summary>

#### Kubernetes

CRI (Container Runtime Interface) — стандартний gRPC-інтерфейс між `kubelet` і
container runtime.

Завдяки CRI Kubernetes абстрагує запуск контейнерів від конкретної реалізації
runtime.

Що це дає:

- взаємозамінність runtime без змін у workload YAML;
- єдиний контракт для create/start/stop pod sandbox і контейнерів;
- простіша еволюція платформи без привʼязки до одного runtime.

**Коротко:**

- CRI — це API-шар між Kubernetes і runtime.
- `kubelet` керує контейнерами через CRI, а не напряму через Docker API.
- CRI забезпечує portability між runtime-реалізаціями.

</details>

<details>
<summary>20. Які container runtimes підтримуються (containerd, CRI-O)?</summary>

#### Kubernetes

У production Kubernetes стандартно використовують CRI-сумісні runtimes:
`containerd` і `CRI-O`.

`containerd`:

- найпоширеніший runtime у managed і self-managed кластерах;
- стабільна інтеграція, широка підтримка екосистеми.

`CRI-O`:

- runtime, орієнтований саме на Kubernetes;
- часто використовується у платформах на базі OpenShift/enterprise Linux.

Dockershim видалений із Kubernetes; Docker Engine не є нативним runtime для
`kubelet`.

**Коротко:**

- Актуальні runtimes: `containerd` і `CRI-O`.
- Обидва працюють через CRI та підтримують production-сценарії.
- Docker як runtime для Kubernetes не використовується.

</details>

<details>
<summary>21. Що таке Deployment?</summary>

#### Kubernetes

Deployment — controller для керування stateless workload через декларативні
оновлення Pod.

Deployment керує ReplicaSet і забезпечує:

- потрібну кількість реплік (`spec.replicas`);
- rolling update без повної зупинки сервісу;
- rollback до попередньої ревізії;
- scale up/down через зміну desired state.

Базові команди:

```bash
kubectl get deploy -n app
kubectl apply -f deployment.yaml -n app
kubectl rollout status deploy/api -n app
```

**Коротко:**

- Deployment — стандартний механізм для stateless застосунків.
- Працює через ReplicaSet і revision history.
- Дає безпечні оновлення та відкат змін.

</details>

<details>
<summary>22. Як Deployment виконує rolling update?</summary>

#### Kubernetes

Rolling update замінює Pod поступово, керуючи одночасно старими та новими
репліками.

Ключові параметри стратегії:

- `maxUnavailable` — скільки Pod можна тимчасово втратити;
- `maxSurge` — скільки додаткових Pod можна створити понад ціль;
- probes (`readiness/liveness/startup`) — коли Pod вважається готовим.

При зміні image або Pod template Deployment створює новий ReplicaSet і
перерозподіляє трафік через Service на ready Pod.

```bash
kubectl set image deployment/api api=ghcr.io/org/api:v2 -n app
kubectl rollout status deployment/api -n app
kubectl get rs -n app
```

**Коротко:**

- Rolling update виконує поступову заміну Pod без повного downtime.
- Поведінка оновлення керується `maxUnavailable` і `maxSurge`.
- Readiness probes критичні для zero-downtime.

</details>

<details>
<summary>23. Як виконати rollback Deployment?</summary>

#### Kubernetes

Rollback повертає Deployment до попередньої стабільної ревізії ReplicaSet.

Практичний процес:

```bash
kubectl rollout history deployment/api -n app
kubectl rollout undo deployment/api -n app
kubectl rollout undo deployment/api --to-revision=3 -n app
kubectl rollout status deployment/api -n app
```

Рекомендовано перед rollback перевірити причину деградації через:
`kubectl describe deployment`, `kubectl get events`, `kubectl logs`.

**Коротко:**

- Rollback виконується через `kubectl rollout undo`.
- Можна відкотити до конкретної ревізії.
- Після відкату перевіряють статус rollout і здоровʼя Pod.

</details>

<details>
<summary>24. Що таке ReplicaSet?</summary>

#### Kubernetes

ReplicaSet гарантує, що в кластері постійно працює задана кількість однакових
Pod за selector.

ReplicaSet:

- створює нові Pod, якщо частина впала;
- видаляє зайві Pod, якщо фактичних більше за desired;
- зазвичай керується Deployment, а не напряму.

Пряме ручне керування ReplicaSet рідко використовується у production.

**Коротко:**

- ReplicaSet підтримує стабільну кількість Pod.
- Це базовий механізм self-healing для stateless реплік.
- У більшості випадків ReplicaSet створює й оновлює Deployment.

</details>

<details>
<summary>25. Що таке StatefulSet і коли його використовувати?</summary>

#### Kubernetes

StatefulSet — controller для stateful workload, де важливі стабільна
ідентичність Pod, порядок запуску/зупинки та постійне сховище.

Використання:

- бази даних (PostgreSQL, MongoDB, Cassandra);
- брокери повідомлень і кластери з quorum;
- сервіси, де Pod має стабільне DNS-імʼя (`pod-0`, `pod-1`).

StatefulSet часто працює разом із headless Service і PVC templates.

**Коротко:**

- StatefulSet потрібен для застосунків зі станом і стабільною ідентичністю.
- Дає ordered rollout/termination та стабільні volume привʼязки.
- Типовий вибір для DB і distributed stateful систем.

</details>

<details>
<summary>26. Що таке DaemonSet?</summary>

#### Kubernetes

DaemonSet забезпечує запуск одного Pod на кожному (або вибраних) Node.

Типові сценарії:

- лог-агенти;
- node monitoring/exporters;
- CNI/мережеві агенти;
- security scanning agent на вузлі.

При додаванні нового Node DaemonSet автоматично розгортає на ньому Pod.

**Коротко:**

- DaemonSet = по одному Pod на Node.
- Використовується для node-level інфраструктурних агентів.
- Автоматично масштабується разом із кількістю вузлів.

</details>

<details>
<summary>27. Що таке Job?</summary>

#### Kubernetes

Job — ресурс для одноразових або batch-задач, які мають успішно завершитись.

Job керує:

- кількістю успішних завершень (`completions`);
- рівнем паралелізму (`parallelism`);
- retry-поведінкою (`backoffLimit`).

Приклади: міграції БД, масова обробка даних, генерація звітів.

**Коротко:**

- Job виконує кінцеві задачі до стану `Complete`.
- Підходить для batch і адміністративних операцій.
- Підтримує retry, паралельність і контроль завершення.

</details>

<details>
<summary>28. Що таке CronJob?</summary>

#### Kubernetes

CronJob запускає Job за розкладом (`cron`), наприклад щогодини або раз на добу.

Ключові параметри:

- `schedule` — cron-вираз;
- `concurrencyPolicy` — чи дозволяти паралельні запуски;
- `startingDeadlineSeconds` — вікно пропущеного запуску;
- `successfulJobsHistoryLimit`/`failedJobsHistoryLimit` — історія.

Типові задачі: backup, cleanup, синхронізація, регулярні перевірки.

**Коротко:**

- CronJob = планувальник для періодичних Job.
- Дозволяє контролювати конкуренцію та політику пропущених запусків.
- Основний інструмент для регламентних фонових задач у кластері.

</details>

<details>
<summary>29. Коли використовувати StatefulSet замість Deployment?</summary>

#### Kubernetes

StatefulSet обирають замість Deployment, коли потрібен керований стан і
стабільна ідентичність Pod.

Ознаки, що потрібен StatefulSet:

- Pod має постійний volume, привʼязаний саме до нього;
- важливий порядок старту/зупинки;
- потрібні стабільні мережеві імена між репліками;
- кластерний застосунок має ролі `primary/replica` або quorum.

Якщо сервіс stateless і не потребує стабільних Pod identity/storage, краще
Deployment.

**Коротко:**

- StatefulSet для stateful ідентичних, але не взаємозамінних реплік.
- Deployment для взаємозамінних stateless Pod.
- Ключовий критерій: стан, identity і порядок життєвого циклу.

</details>

<details>
<summary>30. Як розгорнути базу даних у Kubernetes?</summary>

#### Kubernetes

Базу даних у Kubernetes розгортають як Stateful workload із персистентним
storage, політиками безпеки та резервуванням.

Базовий production-підхід:

1. Використати `StatefulSet` + headless Service.
2. Виділити `PVC` через `StorageClass` (dynamic provisioning).
3. Винести конфігурацію в `ConfigMap`, секрети в `Secret`.
4. Додати probes, `requests/limits`, `PodDisruptionBudget`.
5. Налаштувати backup/restore (CronJob або зовнішній оператор).
6. Обмежити доступ через `NetworkPolicy` і `RBAC`.

Короткий приклад перевірки:

```bash
kubectl get statefulset,pvc,pods -n data
kubectl describe statefulset postgres -n data
kubectl logs postgres-0 -n data
```

Для production часто використовують DB Operator (наприклад, PostgreSQL/MongoDB
operator), а не "ручний" StatefulSet.

**Коротко:**

- Для БД потрібні StatefulSet, persistent volumes і контроль доступу.
- Надійність визначають backup/restore, probes і ресурсна ізоляція.
- Для складних кластерів БД краще використовувати Operators.

</details>

<details>
<summary>31. Що таке ConfigMap?</summary>

#### Kubernetes

ConfigMap — ресурс Kubernetes для зберігання несекретної конфігурації у форматі
`key/value`.

ConfigMap використовують для:

- змінних середовища застосунку;
- конфіг-файлів через volume mount;
- розділення конфігурації і container image.

```bash
kubectl create configmap api-config --from-literal=LOG_LEVEL=info -n app
kubectl get configmap api-config -n app -o yaml
```

**Коротко:**

- ConfigMap зберігає несекретну конфігурацію.
- Дозволяє змінювати параметри без перебілду образу.
- Подається в Pod через env або volume.

</details>

<details>
<summary>32. Що таке Secret?</summary>

#### Kubernetes

Secret — ресурс для зберігання чутливих даних: паролів, токенів, API-ключів,
сертифікатів.

Типи Secret: `Opaque`, `kubernetes.io/tls`, `kubernetes.io/dockerconfigjson`
тощо.

Секрети можна передавати в Pod:

- як env vars;
- як файли у volume;
- через CSI secret drivers із зовнішніх vault-систем.

**Коротко:**

- Secret призначений для конфіденційних даних.
- Дає керований спосіб доставки секретів у Pod.
- Має використовуватись разом із RBAC і шифруванням.

</details>

<details>
<summary>33. Як безпечно зберігаються Secrets?</summary>

#### Kubernetes

Безпечне зберігання Secrets базується на кількох шарах контролю.

Практика production:

- увімкнений `encryption at rest` для `etcd`;
- строгий `RBAC` (least privilege) на `secrets`;
- мінімізація використання env vars для секретів;
- аудит доступу до API і ротація секретів;
- інтеграція зі зовнішніми secret manager (Vault, KMS-backed рішення).

```bash
kubectl auth can-i get secrets -n app --as system:serviceaccount:app:api
```

**Коротко:**

- Безпека Secret = encryption + access control + операційна гігієна.
- Найкритичніше: обмежити читання Secret через RBAC.
- У production часто використовують зовнішній secret manager.

</details>

<details>
<summary>34. Що таке encryption at rest для Secrets?</summary>

#### Kubernetes

Encryption at rest — шифрування даних Secret перед записом у `etcd`.

Без цього Secret у `etcd` зберігаються як base64-encoded дані, що не є
криптографічним захистом.

Механіка:

- API server використовує `EncryptionConfiguration`;
- ключі зберігаються через KMS/provider;
- при читанні дані дешифруються прозоро для авторизованих запитів.

**Коротко:**

- Encryption at rest захищає Secret у сховищі `etcd`.
- Base64 саме по собі не є шифруванням.
- Це обовʼязковий контроль для production-кластерів.

</details>

<details>
<summary>35. Як передати конфігурацію в Pod?</summary>

#### Kubernetes

Конфігурацію в Pod передають через `ConfigMap` і `Secret` трьома основними
способами.

1. Env vars (`env`, `envFrom`).
2. Файли через volume mount.
3. Аргументи контейнера (`command/args`) разом із env.

Приклад:

```yaml
envFrom:
  - configMapRef:
      name: api-config
  - secretRef:
      name: api-secrets
```

Для runtime-конфігів, які повинні оновлюватись без redeploy, частіше
використовують mounted files + reload-логіку.

**Коротко:**

- Базові джерела конфігурації: ConfigMap і Secret.
- Найпростіше передавати через env, гнучкіше через volumes.
- Спосіб залежить від вимог до оновлення конфігурації під час роботи.

</details>

<details>
<summary>36. Які best practices для роботи з Secrets?</summary>

#### Kubernetes

Ключові best practices:

- не зберігати секрети у Git у відкритому вигляді;
- обмежити доступ до `Secret` через namespace-scoped RBAC;
- увімкнути encryption at rest і audit logging;
- застосовувати короткоживучі креденшіали та ротацію;
- монтувати секрети у файли, якщо важлива ротація без перезапуску;
- уникати виводу секретів у логах/`kubectl describe`.

У GitOps-сценаріях використовують Sealed Secrets/SOPS або зовнішній secret
operator.

**Коротко:**

- Secret потребують lifecycle-керування, а не лише створення.
- RBAC, ротація і шифрування — мінімальний baseline.
- GitOps для секретів має бути з контрольованим шифруванням.

</details>

<details>
<summary>37. Як працює мережа в Kubernetes?</summary>

#### Kubernetes

Мережа Kubernetes базується на моделі "кожен Pod має власний IP і може
спілкуватися з іншими Pod без NAT між Pod".

Базові рівні:

- CNI забезпечує pod networking;
- Service дає стабільну віртуальну адресу для групи Pod;
- Ingress/Gateway API керують north-south трафіком;
- NetworkPolicy контролює дозволені потоки.

Data path залежить від CNI (iptables/ipvs/eBPF).

**Коротко:**

- Pod-to-Pod, Service і ingress-рівень формують повну мережеву модель.
- CNI визначає реалізацію мережевого dataplane.
- Без NetworkPolicy трафік між Pod зазвичай відкритий за замовчуванням.

</details>

<details>
<summary>38. Що таке Service?</summary>

#### Kubernetes

Service — абстракція стабільної мережевої точки доступу до динамічного набору
Pod (через label selector).

Service вирішує:

- стабільний endpoint незалежно від перезапуску Pod;
- балансування трафіку між ready endpoints;
- service discovery через DNS.

```bash
kubectl get svc -n app
kubectl describe svc api -n app
```

**Коротко:**

- Service дає стабільну адресу для змінних Pod.
- Трафік маршрутизується на healthy endpoints.
- Це базовий механізм внутрішньокластерного доступу.

</details>

<details>
<summary>39. Типи Service: ClusterIP, NodePort, LoadBalancer — у чому різниця?</summary>

#### Kubernetes

`ClusterIP`:

- доступний лише всередині кластера;
- стандартний тип для service-to-service трафіку.

`NodePort`:

- відкриває порт на кожному Node;
- трафік заходить через `<NodeIP>:<NodePort>`.

`LoadBalancer`:

- у cloud-середовищі створює зовнішній балансувальник;
- маршрутизує трафік на Service (зазвичай поверх NodePort/інтеграції провайдера).

**Коротко:**

- `ClusterIP` для внутрішнього доступу.
- `NodePort` для прямого доступу через вузли.
- `LoadBalancer` для керованого зовнішнього доступу.

</details>

<details>
<summary>40. Що таке Headless Service?</summary>

#### Kubernetes

Headless Service — Service з `clusterIP: None`, який не надає єдиного VIP.

DNS повертає IP конкретних Pod endpoints, що корисно для:

- StatefulSet (стабільні pod DNS-імена);
- клієнтського балансування;
- кластерних stateful систем із peer discovery.

**Коротко:**

- Headless Service відключає virtual IP і повертає адреси Pod.
- Часто використовується зі StatefulSet.
- Дає контроль над прямою адресацією реплік.

</details>

<details>
<summary>41. Як працює service discovery?</summary>

#### Kubernetes

Service discovery у Kubernetes реалізується через DNS (CoreDNS) і Service-імена.

Типовий формат DNS-імен:

- `service.namespace.svc.cluster.local`

Коли Pod звертається до `api.app.svc`, DNS повертає Service IP (або Pod IP для
headless Service).

```bash
kubectl get endpoints api -n app
kubectl get endpointslices -n app
```

**Коротко:**

- Service discovery у Kubernetes — DNS-first модель.
- CoreDNS + Service/EndpointSlice забезпечують резолюцію й маршрутизацію.
- Для headless Service повертаються адреси Pod, а не один VIP.

</details>

<details>
<summary>42. Що таке Ingress?</summary>

#### Kubernetes

Ingress — API-ресурс L7 маршрутизації HTTP(S) трафіку до Service за host/path
правилами.

Ingress дозволяє:

- термінацію TLS;
- virtual hosts;
- path-based routing;
- централізований вхідний трафік для кількох сервісів.

Ingress сам по собі не працює без Ingress Controller.

**Коротко:**

- Ingress описує HTTP(S) правила входу в кластер.
- Дає host/path маршрутизацію і TLS на edge-рівні.
- Для роботи потребує встановленого Ingress Controller.

</details>

<details>
<summary>43. Що таке Ingress Controller?</summary>

#### Kubernetes

Ingress Controller — data-plane компонент, який читає ресурси Ingress і
реалізує їх у реальній мережевій конфігурації (Nginx, HAProxy, Envoy тощо).

Функції:

- завантаження правил Ingress;
- маршрутизація до backend Service;
- TLS termination, redirects, rate limiting (залежно від контролера).

Без Ingress Controller обʼєкти Ingress не впливають на трафік.

**Коротко:**

- Ingress Controller виконує правила, описані в Ingress.
- Це runtime-компонент, не лише декларація.
- Вибір контролера впливає на можливості L7 і продуктивність.

</details>

<details>
<summary>44. Що таке Gateway API?</summary>

#### Kubernetes

Gateway API — сучасний набір Kubernetes API для L4/L7 трафіку з чітким
розділенням ролей платформи і застосунку.

Основні ресурси:

- `GatewayClass` — клас/реалізація контролера;
- `Gateway` — мережевий вхідний шлюз (listeners, TLS, адресація);
- `HTTPRoute`/`GRPCRoute`/інші Route — правила маршрутизації.

Gateway API дає кращу розширюваність і портативність, ніж класичний Ingress.

**Коротко:**

- Gateway API — наступний етап розвитку ingress-моделі в Kubernetes.
- Дає типізовані маршрути та кращу модель delegation.
- Підходить для складних multi-team edge сценаріїв.

</details>

<details>
<summary>45. Чим Gateway API відрізняється від Ingress?</summary>

#### Kubernetes

Головні відмінності:

- Ingress: один базовий ресурс з обмеженою моделлю;
- Gateway API: набір ресурсів з окремими ролями (infra vs app teams);
- Gateway API має багатші policy/route моделі і кращу extensibility.

Практично це означає:

- гнучкіше керування listener-ами і маршрутами;
- безпечніша делегація маршрутів між namespace;
- зручніша підтримка не лише HTTP use-cases.

**Коротко:**

- Ingress простіший, Gateway API масштабованіший по архітектурі.
- Gateway API краще для великих платформ і складного трафік-менеджменту.
- Обидва потребують відповідного контролера даних.

</details>

<details>
<summary>46. Що таке Gateway, GatewayClass та HTTPRoute?</summary>

#### Kubernetes

`GatewayClass` — описує, який контролер/платформний клас обслуговує Gateway.

`Gateway`:

- визначає точки входу (listeners), порти, TLS і привʼязку до інфраструктури;
- зазвичай управляється platform team.

`HTTPRoute`:

- описує host/path/match правила та backend refs;
- зазвичай управляється командою застосунку.

Разом вони реалізують розділення відповідальності edge-інфраструктури і
маршрутів застосунків.

**Коротко:**

- `GatewayClass` = тип реалізації.
- `Gateway` = вхідний шлюз і listeners.
- `HTTPRoute` = правила маршрутизації до сервісів.

</details>

<details>
<summary>47. Що таке NetworkPolicy?</summary>

#### Kubernetes

NetworkPolicy — ресурс для контролю мережевого доступу до/від Pod на рівні
L3/L4 (IP, порт, протокол).

Дозволяє описувати:

- `ingress` правила до Pod;
- `egress` правила з Pod;
- селекцію за labels namespace/pod.

Працює лише якщо CNI підтримує NetworkPolicy enforcement.

**Коротко:**

- NetworkPolicy обмежує мережеву взаємодію між workload.
- Дозволяє будувати мікросегментацію в межах кластера.
- Ефективність залежить від можливостей CNI.

</details>

<details>
<summary>48. Як NetworkPolicy підвищує безпеку?</summary>

#### Kubernetes

NetworkPolicy реалізує принцип least privilege для мережі: дозволяється лише
потрібний трафік.

Безпековий ефект:

- зменшення lateral movement у разі компрометації Pod;
- ізоляція чутливих сервісів (DB, auth, internal API);
- контроль outbound доступу до зовнішніх endpoint.

Практичний підхід: default-deny + явні allow-правила.

**Коротко:**

- NetworkPolicy сегментує трафік і зменшує blast radius.
- Найкраща практика: починати з deny-by-default.
- Це базовий елемент defense-in-depth у Kubernetes.

</details>

<details>
<summary>49. Що таке Persistent Volume (PV)?</summary>

#### Kubernetes

PV — кластерний ресурс, що представляє виділене сховище (block/file) з
визначеними параметрами доступу і життєвого циклу.

PV може бути:

- static (створений адміністратором);
- dynamic (створений автоматично через StorageClass).

PV існує незалежно від конкретного Pod.

**Коротко:**

- PV описує реальний storage-ресурс у кластері.
- Відокремлює життєвий цикл даних від Pod.
- Використовується через механізм PVC binding.

</details>

<details>
<summary>50. Що таке Persistent Volume Claim (PVC)?</summary>

#### Kubernetes

PVC — запит застосунку на сховище з певним розміром, режимом доступу і
класом storage.

Pod не монтує PV напряму: він монтує PVC, який привʼязується до сумісного PV.

```bash
kubectl get pvc -n data
kubectl describe pvc pgdata-postgres-0 -n data
```

**Коротко:**

- PVC — це декларація потреби застосунку у persistent storage.
- Kubernetes автоматично підбирає/створює відповідний PV.
- Pod працює з PVC як стабільною абстракцією storage.

</details>

<details>
<summary>51. У чому різниця між PV і PVC?</summary>

#### Kubernetes

`PV` — це ресурс постачання сховища (supply side),  
`PVC` — це запит споживача на сховище (demand side).

Модель схожа на "ресурс і контракт його споживання":

- адміністратор/платформа надає класи та backend;
- workload-команда описує потребу через PVC;
- control plane виконує binding між PVC і PV.

**Коротко:**

- PV = що кластер може надати.
- PVC = що застосунок запитує.
- Звʼязування відбувається автоматично за сумісними параметрами.

</details>

<details>
<summary>52. Що таке StorageClass?</summary>

#### Kubernetes

StorageClass описує "профіль" сховища для dynamic provisioning: тип диска,
параметри, reclaim policy, binding mode.

Що задає StorageClass:

- provisioner (CSI driver);
- performance/availability параметри;
- `volumeBindingMode` (`Immediate`/`WaitForFirstConsumer`);
- `reclaimPolicy` (`Delete`/`Retain`).

**Коротко:**

- StorageClass стандартизує класи storage для команд.
- Керує тим, як і де створюються volumes.
- Є основою dynamic provisioning у Kubernetes.

</details>

<details>
<summary>53. Що таке dynamic provisioning?</summary>

#### Kubernetes

Dynamic provisioning — автоматичне створення PV у момент створення PVC за
правилами StorageClass.

Переваги:

- без ручного створення PV під кожен workload;
- швидше розгортання stateful сервісів;
- узгоджені параметри storage через стандартизовані класи.

Це типовий production-підхід у cloud і сучасних on-prem кластерах.

**Коротко:**

- PVC тригерить автоматичне створення PV через StorageClass.
- Dynamic provisioning прибирає ручні storage-операції.
- Дає масштабовану модель роботи з persistent volumes.

</details>

<details>
<summary>54. Як StatefulSet працює з постійним сховищем?</summary>

#### Kubernetes

StatefulSet використовує `volumeClaimTemplates`, щоб для кожної репліки
створювався окремий PVC.

Приклади імен:

- `data-mydb-0`, `data-mydb-1` для `mydb-0`, `mydb-1`.

Це гарантує:

- стабільну привʼязку сховища до конкретної репліки;
- збереження даних при пересозданні Pod;
- передбачувану identity + storage модель для stateful кластерів.

**Коротко:**

- StatefulSet створює індивідуальний PVC на кожен Pod.
- Репліка повторно піднімається зі своїм volume.
- Така модель критична для БД і quorum-систем.

</details>

<details>
<summary>55. Що відбувається з даними після перезапуску Pod?</summary>

#### Kubernetes

Залежить від типу сховища:

- дані в writable layer контейнера зникають при пересозданні Pod;
- дані в `emptyDir` живуть лише поки Pod існує;
- дані на `PVC/PV` зберігаються і підмонтовуються після restart/recreate.

Тому для стану застосунку (БД, черги, файли) потрібен persistent volume.

**Коротко:**

- Ефемерне сховище Pod не гарантує збереження даних.
- Лише PVC/PV дають персистентність між перезапусками.
- Для stateful workload persistent storage є обовʼязковим.

</details>
<details>
<summary>56. Як scheduler обирає Node?</summary>

#### Kubernetes

`kube-scheduler` фільтрує і ранжує вузли, а потім робить binding Pod до
найкращого Node.

Етапи:

- filtering: перевірка ресурсів, taints/tolerations, affinity, topology;
- scoring: пріоритети (баланс, locality, anti-affinity, spread);
- binding: запис призначення Pod -> Node в API.

**Коротко:**

- Scheduler не запускає контейнери, а лише обирає вузол.
- Вибір базується на політиках і доступних ресурсах.
- Після binding Pod піднімає `kubelet` на вибраному Node.

</details>

<details>
<summary>57. Що таке resource requests і limits?</summary>

#### Kubernetes

`requests` — гарантований мінімум ресурсу для планування Pod.
`limits` — верхня межа використання ресурсу контейнером.

Для CPU і RAM:

- scheduler враховує саме `requests`;
- перевищення `memory limit` призводить до OOMKill;
- CPU при перевищенні `limit` throttling-иться.

**Коротко:**

- `requests` впливають на placement Pod.
- `limits` обмежують споживання під час виконання.
- Коректні значення критичні для стабільності кластера.

</details>

<details>
<summary>58. Що таке QoS (Guaranteed, Burstable, BestEffort)?</summary>

#### Kubernetes

QoS клас визначає пріоритет виживання Pod під ресурсним тиском.

- `Guaranteed`: у всіх контейнерів `requests == limits` для CPU/RAM.
- `Burstable`: є `requests`, але `requests != limits` хоча б десь.
- `BestEffort`: немає `requests` і `limits`.

При eviction найменш пріоритетний — `BestEffort`.

**Коротко:**

- QoS визначає, які Pod видалятимуться першими при дефіциті ресурсів.
- `Guaranteed` має найвищий пріоритет стабільності.
- QoS напряму залежить від requests/limits.

</details>

<details>
<summary>59. Що таке nodeSelector?</summary>

#### Kubernetes

`nodeSelector` — простий механізм привʼязки Pod до Node за label exact-match.

Приклад:

```yaml
nodeSelector:
  nodepool: payments
```

Якщо жоден Node не відповідає label, Pod залишиться в `Pending`.

**Коротко:**

- `nodeSelector` — найпростіший спосіб node placement.
- Працює лише як точний збіг label.
- Для складних правил краще `nodeAffinity`.

</details>

<details>
<summary>60. Що таке node affinity?</summary>

#### Kubernetes

`nodeAffinity` — розширений механізм планування Pod на вузли за label-правилами.

Режими:

- `requiredDuringSchedulingIgnoredDuringExecution` — жорстка вимога;
- `preferredDuringSchedulingIgnoredDuringExecution` — мʼяке побажання.

Підтримує оператори `In`, `NotIn`, `Exists`, `DoesNotExist`.

**Коротко:**

- Node affinity гнучкіший за `nodeSelector`.
- Дозволяє hard/soft правила placement.
- Використовується для зон, типів вузлів і спецпулів.

</details>

<details>
<summary>61. Що таке pod affinity та anti-affinity?</summary>

#### Kubernetes

Pod affinity/anti-affinity керує взаємним розміщенням Pod відносно інших Pod.

- affinity: розміщувати поруч із певними Pod;
- anti-affinity: не розміщувати поруч.

Часто застосовується з `topologyKey` (`kubernetes.io/hostname`, zone).

**Коротко:**

- Pod affinity керує co-location workload.
- Pod anti-affinity підвищує відмовостійкість через розподіл реплік.
- Це важливий інструмент для HA-дизайну.

</details>

<details>
<summary>62. Що таке taints і tolerations?</summary>

#### Kubernetes

Taint ставиться на Node і "відштовхує" Pod.
Toleration у Pod дозволяє ігнорувати конкретний taint.

Типовий сценарій: виділити вузли під спецнавантаження (GPU, infra, DB).

Ефекти taint:

- `NoSchedule`
- `PreferNoSchedule`
- `NoExecute`

**Коротко:**

- Taints керують тим, кого не можна садити на Node.
- Tolerations не гарантує placement, а лише дозволяє його.
- Використовуються для ізоляції й контрольованого доступу до нод.

</details>

<details>
<summary>63. Як ізолювати навантаження на окремих нодах?</summary>

#### Kubernetes

Комбінують кілька механізмів scheduling:

- окремий node pool із labels;
- `taints` на вузлах + `tolerations` у потрібних Pod;
- `nodeAffinity/nodeSelector` для жорсткої привʼязки;
- `ResourceQuota` і `PriorityClass` для контролю конкуренції.

Для критичних сервісів додають anti-affinity і PDB.

**Коротко:**

- Найнадійніше: taints+tolerations + affinity + виділений node pool.
- Це зменшує noisy-neighbor ефект.
- Ізоляцію треба поєднувати з ресурсними політиками.

</details>

<details>
<summary>64. Що таке Horizontal Pod Autoscaler (HPA)?</summary>

#### Kubernetes

HPA автоматично змінює кількість реплік workload (`Deployment`, `StatefulSet`)
за метриками навантаження.

Типові метрики:

- CPU/Memory utilization;
- custom/external metrics (через adapter).

**Коротко:**

- HPA масштабує кількість Pod горизонтально.
- Працює за поточним навантаженням.
- Основний autoscaling-механізм для додатків.

</details>

<details>
<summary>65. Як працює HPA?</summary>

#### Kubernetes

Контролер HPA періодично читає метрики, обчислює бажану кількість реплік і
оновлює `scale` target-ресурсу.

Базова логіка:

- поточна метрика > target -> scale up;
- поточна метрика < target -> scale down (з урахуванням stabilization window).

Для CPU/memory зазвичай потрібен Metrics Server.

**Коротко:**

- HPA — це control loop над метриками.
- Скейлить через оновлення `spec.replicas` у target workload.
- Параметри поведінки важливі для уникнення "флапінгу".

</details>

<details>
<summary>66. Що таке Metrics Server і навіщо він потрібен?</summary>

#### Kubernetes

Metrics Server збирає базові ресурсні метрики Pod/Node (CPU, memory) для
Kubernetes autoscaling API.

Використовується для:

- HPA на CPU/memory;
- `kubectl top pods/nodes`.

Це не повноцінний monitoring stack (не замінює Prometheus).

**Коротко:**

- Metrics Server дає оперативні ресурсні метрики для autoscaling.
- Потрібен для типових HPA сценаріїв.
- Для історії/алертів потрібен окремий стек моніторингу.

</details>

<details>
<summary>67. Що таке Vertical Pod Autoscaler (VPA)?</summary>

#### Kubernetes

VPA автоматично підбирає `requests/limits` для контейнерів на основі історії
споживання ресурсів.

Режими:

- `Off` (recommendation only);
- `Initial` (на старті Pod);
- `Auto`/`Recreate` (оновлення через перезапуск Pod).

**Коротко:**

- VPA масштабує ресурси вертикально, а не кількість реплік.
- Допомагає прибрати over/under-provisioning.
- Часто використовується окремо від HPA на CPU.

</details>

<details>
<summary>68. Що таке Cluster Autoscaler?</summary>

#### Kubernetes

Cluster Autoscaler масштабує кількість Node у кластері за станом scheduler.

Поведінка:

- додає вузли, коли Pod не можуть бути заплановані;
- прибирає недовантажені вузли, якщо Pod можна безпечно перенести.

Працює через інтеграцію з cloud/node group API.

**Коротко:**

- Cluster Autoscaler масштабує інфраструктуру, не Pod.
- Закриває дефіцит або надлишок вузлів.
- Працює в парі з HPA/VPA для повного autoscaling.

</details>

<details>
<summary>69. У чому різниця між HPA, VPA та Cluster Autoscaler?</summary>

#### Kubernetes

- HPA: змінює кількість Pod (`replicas`).
- VPA: змінює ресурси Pod (`requests/limits`).
- Cluster Autoscaler: змінює кількість Node.

Разом вони працюють на різних рівнях: workload, pod sizing, cluster capacity.

**Коротко:**

- HPA = горизонтальне масштабування застосунку.
- VPA = вертикальне тюнінгування ресурсів Pod.
- CA = масштабування обчислювальної бази кластера.

</details>

<details>
<summary>70. Як масштабувати додаток за кастомними метриками?</summary>

#### Kubernetes

Для custom metrics HPA використовує API `custom.metrics.k8s.io` або
`external.metrics.k8s.io` через adapter (часто Prometheus Adapter).

Типовий потік:

1. Експортувати метрику (наприклад, queue lag, RPS).
2. Зібрати її в Prometheus.
3. Віддати в HPA через adapter.
4. Описати target в `autoscaling/v2` HPA.

**Коротко:**

- Кастомні метрики дають бізнес-орієнтований autoscaling.
- Найчастіше реалізується через Prometheus Adapter.
- Потрібен контроль якості метрик, щоб уникнути нестабільного scaling.

</details>

<details>
<summary>71. Що таке RollingUpdate?</summary>

#### Kubernetes

RollingUpdate — стратегія поступового оновлення Pod без повної зупинки сервісу.

У `Deployment` контролюється через:

- `maxUnavailable`
- `maxSurge`

Трафік іде на готові Pod за readiness probes.

**Коротко:**

- RollingUpdate мінімізує downtime під час релізу.
- Оновлення виконується поетапно.
- Це дефолтна і найбільш практична стратегія для stateless сервісів.

</details>

<details>
<summary>72. Що таке стратегія Recreate?</summary>

#### Kubernetes

`Recreate` спочатку зупиняє старі Pod, а потім запускає нові.

Коли доречно:

- додаток не підтримує паралельну роботу старої/нової версії;
- є жорсткі несумісності стану або схеми.

Ризик: повний або частковий downtime на час заміни.

**Коротко:**

- Recreate = stop old -> start new.
- Простий, але ризиковий щодо доступності.
- Використовується лише коли RollingUpdate неможливий.

</details>

<details>
<summary>73. Як реалізувати Blue/Green deployment?</summary>

#### Kubernetes

Створюють два оточення однієї версії сервісу: `blue` (active) і `green`
(candidate), після перевірки перемикають трафік.

Реалізація:

- два Deployment з різними labels;
- один Service, який перемикає selector;
- або керування через Ingress/Gateway weights.

**Коротко:**

- Blue/Green дає швидкий cutover і простий rollback.
- Потребує подвійних ресурсів на час релізу.
- Добре підходить для критичних оновлень.

</details>

<details>
<summary>74. Як реалізувати Canary deployment?</summary>

#### Kubernetes

Canary подає малу частину трафіку на нову версію і поступово збільшує відсоток
після перевірки метрик.

Реалізація:

- окремі stable/canary Deployment;
- маршрутизація через Ingress/Gateway/service mesh;
- автоматичний промоушн/rollback за SLO.

**Коротко:**

- Canary зменшує ризик релізу через поетапне введення версії.
- Рішення про промоушн базується на метриках, не на таймері.
- Потребує інструментів traffic splitting і observability.

</details>

<details>
<summary>75. Які інструменти використовуються для progressive delivery (Argo Rollouts, Flagger)?</summary>

#### Kubernetes

Найуживаніші інструменти:

- Argo Rollouts: canary/blue-green controller з analysis і rollback;
- Flagger: автоматизує progressive delivery на базі метрик;
- service mesh/ingress інтеграції (Istio, Linkerd, NGINX, Gateway API).

**Коротко:**

- Progressive delivery потребує окремого rollout-контролера.
- Argo Rollouts і Flagger — стандартні production-варіанти.
- Ключова умова: надійні метрики та automated rollback.

</details>

<details>
<summary>76. Як забезпечити zero-downtime deployment?</summary>

#### Kubernetes

Потрібна комбінація deployment і runtime-практик:

- RollingUpdate з адекватними `maxUnavailable/maxSurge`;
- коректні `readiness`/`startup` probes;
- graceful shutdown (`preStop`, `terminationGracePeriodSeconds`);
- кілька реплік + anti-affinity;
- PDB і контроль disruption.

**Коротко:**

- Zero-downtime досягається політикою оновлення + готовністю Pod.
- Readiness probe критичні для безпечного переключення трафіку.
- Один Pod або відсутність grace shutdown зазвичай ламають ціль.

</details>

<details>
<summary>77. Що таке Pod Disruption Budget (PDB)?</summary>

#### Kubernetes

PDB обмежує кількість Pod, які можна добровільно евакуювати одночасно
(maintenance, drain, autoscaler scale-down).

Параметри:

- `minAvailable`
- `maxUnavailable`

PDB не захищає від аварійного падіння ноди/процесу.

**Коротко:**

- PDB контролює voluntary disruptions.
- Допомагає зберегти мінімальну доступність сервісу.
- Працює разом із rollout/disruption процесами.

</details>

<details>
<summary>78. Чому PDB важливий?</summary>

#### Kubernetes

Без PDB платформа може одночасно видалити занадто багато реплік під час
обслуговування вузлів.

Наслідки відсутності PDB:

- деградація або недоступність сервісу;
- збої під час node upgrade/drain;
- нестабільний autoscaler scale-down.

**Коротко:**

- PDB — страховка доступності під час планових змін.
- Особливо критичний для stateful і latency-sensitive сервісів.
- Є базовою вимогою production-надійності.

</details>

<details>
<summary>79. Що таке RBAC?</summary>

#### Kubernetes

RBAC (Role-Based Access Control) — модель авторизації Kubernetes, яка визначає,
хто і які API-дії може виконувати.

Сутності:

- Role/ClusterRole — набір дозволів;
- RoleBinding/ClusterRoleBinding — привʼязка дозволів до subject.

**Коротко:**

- RBAC керує доступом до ресурсів API.
- Дозволяє реалізувати least privilege.
- Це базовий security-control у Kubernetes.

</details>

<details>
<summary>80. Різниця між Role і ClusterRole?</summary>

#### Kubernetes

- `Role` діє в межах одного namespace.
- `ClusterRole` діє на рівні всього кластера або для non-namespaced ресурсів.

`ClusterRole` можна також привʼязати в конкретному namespace через
`RoleBinding`.

**Коротко:**

- Role = namespace scope.
- ClusterRole = cluster scope.
- Вибір залежить від області доступу.

</details>

<details>
<summary>81. Що таке RoleBinding і ClusterRoleBinding?</summary>

#### Kubernetes

Це ресурси привʼязки прав до користувачів, груп або service accounts.

- `RoleBinding`: надає Role/ClusterRole в межах namespace.
- `ClusterRoleBinding`: надає ClusterRole на весь кластер.

**Коротко:**

- Binding зʼєднує "хто" і "що дозволено".
- RoleBinding обмежує доступ namespace-рамками.
- ClusterRoleBinding застосовується для глобальних прав.

</details>

<details>
<summary>82. Що таке Pod Security Standards?</summary>

#### Kubernetes

Pod Security Standards (PSS) — набір профілів безпеки Pod:

- `Privileged`
- `Baseline`
- `Restricted`

Вони визначають вимоги до security-параметрів контейнерів і Pod.

**Коротко:**

- PSS — стандартизовані рівні жорсткості безпеки Pod.
- У production зазвичай ціль — `Restricted` (з винятками).
- Це policy-модель, яку застосовує Pod Security Admission.

</details>

<details>
<summary>83. Що таке Pod Security Admission?</summary>

#### Kubernetes

Pod Security Admission (PSA) — вбудований admission controller, який перевіряє
Pod проти PSS-політик на рівні namespace labels.

Режими:

- `enforce`
- `warn`
- `audit`

PSP застарів і видалений; актуальний механізм — PSA.

**Коротко:**

- PSA забезпечує policy enforcement для безпеки Pod.
- Керується namespace labels і режимами enforce/warn/audit.
- Це стандартна заміна PodSecurityPolicy.

</details>

<details>
<summary>84. Як запускати контейнери без root?</summary>

#### Kubernetes

Налаштовують `securityContext` контейнера/Pod:

- `runAsNonRoot: true`
- `runAsUser`, `runAsGroup`
- read-only root filesystem за потреби.

Також образ має підтримувати non-root користувача.

**Коротко:**

- Non-root запуск — базова вимога hardening.
- Потрібні і правильний image, і правильний securityContext.
- Це знижує ризик privilege escalation.

</details>

<details>
<summary>85. Що таке SecurityContext?</summary>

#### Kubernetes

`securityContext` — набір security-параметрів для Pod/контейнера:

- user/group IDs;
- capabilities;
- `allowPrivilegeEscalation`;
- `readOnlyRootFilesystem`;
- seccomp, SELinux/AppArmor (де підтримується).

**Коротко:**

- SecurityContext керує привілеями процесів у контейнері.
- Це ключовий механізм runtime hardening.
- Налаштовується на рівні Pod і контейнера.

</details>

<details>
<summary>86. Як обмежити Linux capabilities контейнера?</summary>

#### Kubernetes

У `securityContext.capabilities` прибирають усе зайве і додають лише необхідне.

Практика:

- `drop: ["ALL"]`
- `add` тільки точково (наприклад `NET_BIND_SERVICE`).

Це мінімізує surface для контейнерних атак.

**Коротко:**

- Починати варто з `drop ALL`.
- Додавати тільки capability, без якої сервіс не працює.
- Це важливий елемент least privilege на рівні процесу.

</details>

<details>
<summary>87. Як захистити Kubernetes API server?</summary>

#### Kubernetes

Базовий hardening API server:

- приватний endpoint або жорсткий network allowlist;
- mTLS між компонентами control plane;
- OIDC/корпоративна ідентифікація + MFA на admin-доступ;
- строгий RBAC, мінімум `cluster-admin`;
- admission policies, audit logging, своєчасні оновлення.

**Коротко:**

- API server — найкритичніша точка атаки в кластері.
- Захист будується комбінацією мережі, IAM і політик.
- Регулярний patching і аудит обовʼязкові.

</details>

<details>
<summary>88. Як виконувати аудит дій у кластері?</summary>

#### Kubernetes

Увімкнути Kubernetes audit logging на API server і відправляти логи в
централізоване сховище (SIEM/лог-платформа).

Що аудіювати:

- доступ до `secrets`, RBAC, authn/authz зміни;
- create/update/delete критичних ресурсів;
- exec/port-forward у production namespaces.

**Коротко:**

- Аудит у Kubernetes робиться через audit policy API server.
- Логи мають централізовано зберігатися і корелюватися.
- Без аудиту складно розслідувати інциденти і порушення доступу.

</details>

<details>
<summary>89. Які основні best practices безпеки Kubernetes?</summary>

#### Kubernetes

Ключовий security baseline:

- RBAC least privilege + окремі service accounts;
- PSA (`Restricted`) і securityContext non-root;
- NetworkPolicy default-deny;
- Secrets encryption at rest + ротація;
- image signing/scanning та заборона latest-tags;
- регулярні оновлення Kubernetes і вузлів.

**Коротко:**

- Безпека Kubernetes багатошарова: IAM, мережа, runtime, supply chain.
- Найкраща стратегія — policy-by-default, винятки точково.
- Автоматизація перевірок важливіша за ручні ревʼю.

</details>

<details>
<summary>90. Як переглянути логи Pod?</summary>

#### Kubernetes

Базові команди:

```bash
kubectl logs pod-name -n app
kubectl logs pod-name -c container-name -n app
kubectl logs deploy/api -n app --since=30m
kubectl logs pod-name -n app --previous
```

`--previous` корисний після restart контейнера.

**Коротко:**

- `kubectl logs` — перший крок діагностики Pod.
- Для multi-container Pod потрібно вказувати `-c`.
- `--previous` допомагає аналізувати попередній crash.

</details>

<details>
<summary>91. Як діагностувати CrashLoopBackOff?</summary>

#### Kubernetes

Послідовність перевірки:

1. `kubectl describe pod ...` — reason, exit code, events.
2. `kubectl logs ... --previous` — лог перед падінням.
3. Перевірити probes, env/config, dependencies (DB, DNS, secrets).
4. Перевірити OOMKill/ресурси (`kubectl top`).

**Коротко:**

- CrashLoopBackOff — симптом, а не першопричина.
- Ключові джерела: describe, previous logs, events.
- Часті причини: помилка старту, bad config, OOM, probe misconfig.

</details>

<details>
<summary>92. Як знайти причину постійного перезапуску Pod?</summary>

#### Kubernetes

Перезапуски найчастіше викликають: OOM, failing probes, process crash,
зовнішні залежності.

Що перевірити:

- `restartCount`, `lastState`, `exitCode` у `kubectl describe pod`;
- `kubectl logs --previous`;
- readiness/liveness/startup probes;
- requests/limits і реальне споживання.

**Коротко:**

- Аналіз починається з `describe` і `--previous logs`.
- Причина зазвичай в процесі, конфігурації або ресурсах.
- Правильні probes і sizing суттєво знижують restart-loop.

</details>

<details>
<summary>93. Як діагностувати мережеві проблеми?</summary>

#### Kubernetes

Практичний чеклист:

- перевірити Service/Endpoints/EndpointSlice;
- перевірити DNS (`nslookup`, `dig` з debug pod);
- перевірити NetworkPolicy;
- перевірити Ingress/Gateway маршрути і backend health;
- перевірити CNI/kube-proxy стан на вузлах.

```bash
kubectl get svc,endpoints,endpointslices -n app
kubectl get networkpolicy -n app
kubectl exec -it debug -- nslookup api.app.svc
```

**Коротко:**

- Мережеву проблему локалізують по шарах: DNS -> Service -> Policy -> CNI.
- Endpoints показують, чи бачить Service реальні Pod.
- Debug pod з мережевими утилітами істотно прискорює діагностику.

</details>

<details>
<summary>94. Як переглянути події (events) у кластері?</summary>

#### Kubernetes

Події показують причини змін стану ресурсів і помилки scheduler/kubelet.

```bash
kubectl get events -A --sort-by=.lastTimestamp
kubectl get events -n app --field-selector involvedObject.name=api-xxx
kubectl describe pod api-xxx -n app
```

Events мають короткий retention, тому для production потрібне централізоване
збирання.

**Коротко:**

- Events — швидкий спосіб побачити "що пішло не так".
- Особливо корисні для Pending/Failed scheduling і image pull проблем.
- Їх варто збирати централізовано через короткий TTL.

</details>

<details>
<summary>95. Як моніторити Kubernetes за допомогою Prometheus?</summary>

#### Kubernetes

Типовий стек: Prometheus + kube-state-metrics + node-exporter + Alertmanager +
Grafana.

Що моніторять:

- стан Pod/Deployment/Node;
- ресурси CPU/RAM/disk/network;
- SLI/SLO сервісів;
- control plane метрики.

Найчастіше розгортають через `kube-prometheus-stack` Helm chart.

**Коротко:**

- Prometheus — стандарт де-факто для метрик Kubernetes.
- Потрібні як інфраструктурні, так і прикладні метрики.
- Цінність моніторингу визначається якістю алертів і дашбордів.

</details>

<details>
<summary>96. Що таке kube-state-metrics?</summary>

#### Kubernetes

kube-state-metrics експортує метрики стану Kubernetes-обʼєктів з API server
(не ресурсне використання процесів).

Приклади:

- кількість desired/available реплік;
- статус Pod/Job;
- фази PVC/PV.

**Коротко:**

- kube-state-metrics = метрики стану обʼєктів control plane.
- Доповнює node/cAdvisor метрики ресурсного споживання.
- Критичний для алертів на деградацію workload.

</details>

<details>
<summary>97. Як організувати централізований логінг (EFK / Loki)?</summary>

#### Kubernetes

Патерн: агент на ноді збирає stdout/stderr контейнерів і відправляє в централізоване сховище.

Варіанти:

- EFK: Fluent Bit/Fluentd -> Elasticsearch/OpenSearch -> Kibana;
- Loki stack: Promtail/Fluent Bit -> Loki -> Grafana.

Рекомендації:

- структурувати логи в JSON;
- додавати labels (namespace, app, pod);
- налаштувати retention і контроль вартості.

**Коротко:**

- Централізований логінг потрібен для розслідувань і аудиту.
- Loki зазвичай дешевший для log-at-scale, EFK гнучкіший у full-text.
- Важлива стандартизація формату логів і labels.

</details>

<details>
<summary>98. Як налаштувати алерти (Alertmanager)?</summary>

#### Kubernetes

Prometheus обчислює alert rules, Alertmanager групує/дедуплікує та надсилає
сповіщення (Slack, PagerDuty, email).

Практика:

- алерти за симптомами + алерти за причинами;
- чіткі severity і маршрутизація по командах;
- inhibition/silence під maintenance windows.

**Коротко:**

- Alertmanager керує життєвим циклом сповіщень.
- Якісні алерти мають низький шум і чіткі runbook-посилання.
- Routing і dedup критичні для on-call ефективності.

</details>

<details>
<summary>99. Як знайти причину високого використання ресурсів?</summary>

#### Kubernetes

Порядок аналізу:

1. Знайти проблемний рівень: node, namespace, workload, pod/container.
2. Зіставити `requests/limits` і реальне споживання.
3. Перевірити throttling, OOM, GC, I/O wait, queue lag.
4. Перевірити зміни релізу і трафіку.

Інструменти: `kubectl top`, Prometheus, профілювання застосунку.

**Коротко:**

- Причину шукають зверху вниз: кластер -> workload -> контейнер.
- Часто проблема в неправильному sizing або регресії релізу.
- Метрики інфри і аплікації потрібно аналізувати разом.

</details>

<details>
<summary>100. Різниця між imperative і declarative підходами?</summary>

#### Kubernetes

Imperative: команди типу "зроби дію зараз" (`kubectl scale`, `kubectl set image`).
Declarative: опис цільового стану в YAML + `kubectl apply`.

Declarative кращий для відтворюваності, ревʼю та GitOps.

**Коротко:**

- Imperative зручний для оперативних дій.
- Declarative — база для стабільного production-управління.
- У довгому циклі переважає declarative модель.

</details>

<details>
<summary>101. Що таке declarative configuration?</summary>

#### Kubernetes

Declarative configuration — це опис бажаного стану ресурсів у маніфестах,
який Kubernetes постійно приводить до фактичного.

Основний інструмент:

```bash
kubectl apply -f k8s/
```

**Коротко:**

- Описується "що має бути", а не "як виконати кроки".
- Полегшує контроль змін, відкат і автоматизацію.
- Є фундаментом GitOps-процесів.

</details>

<details>
<summary>102. Що таке GitOps?</summary>

#### Kubernetes

GitOps — операційна модель, у якій Git-репозиторій є джерелом правди для
кластерного стану, а контролер синхронізує кластер із Git.

Принципи:

- всі зміни через pull request;
- автоматична синхронізація і reconcile;
- видимий audit trail змін.

**Коротко:**

- GitOps переносить керування кластером у контрольований Git workflow.
- Зменшує конфіг-дрифт і ручні зміни "в обхід".
- Дає прозорість і відтворюваність операцій.

</details>

<details>
<summary>103. Як працюють Argo CD або Flux?</summary>

#### Kubernetes

Argo CD і Flux запускають у кластері контролери, які:

- читають маніфести з Git;
- порівнюють desired state з live state;
- застосовують зміни та сигналізують про drift.

Підтримуються auto-sync, rollback/pinning і multi-env workflows.

**Коротко:**

- Це reconciliation controllers для GitOps.
- Вони автоматизують deploy і контроль відповідності Git->cluster.
- Різниця переважно у UX та операційній моделі, не у базовому принципі.

</details>

<details>
<summary>104. Чому Git використовується як source of truth?</summary>

#### Kubernetes

Git дає версіонування, review, історію, контроль доступу і простий rollback.

Переваги для платформи:

- прозорі зміни через PR;
- відтворюваність середовищ;
- аудит хто/коли/що змінив.

**Коротко:**

- Git забезпечує керованість конфігурації як коду.
- Це зменшує ризик несанкціонованих і невидимих змін.
- Source of truth у Git спрощує disaster recovery.

</details>

<details>
<summary>105. Як структурувати маніфести для GitOps?</summary>

#### Kubernetes

Практична структура:

- `apps/` для сервісів;
- `platform/` для shared-компонентів;
- `environments/` (`dev/stage/prod`) з overlays/values;
- окремі репо або чіткі каталоги для доступів команд.

Рекомендується: один спосіб шаблонізації (Helm або Kustomize) на один контур.

**Коротко:**

- Структура має відображати ownership і середовища.
- Мінімізуйте дублювання через overlays/чарти.
- Проста ієрархія важливіша за "ідеальну" складність.

</details>

<details>
<summary>106. Що таке drift detection?</summary>

#### Kubernetes

Drift detection — виявлення розходження між станом у Git і фактичним станом у
кластері.

Причини drift:

- ручні `kubectl edit/patch`;
- зовнішні controllers або ad-hoc зміни;
- неповне покриття ресурсів GitOps-контролером.

**Коротко:**

- Drift = кластер відхилився від source of truth.
- GitOps-контролери мають виявляти і виправляти такі відхилення.
- Без drift detection важко гарантувати консистентність середовищ.

</details>

<details>
<summary>107. Що таке Helm?</summary>

#### Kubernetes

Helm — пакетний менеджер Kubernetes для шаблонізації, версіонування і
встановлення наборів маніфестів.

Основні операції:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm upgrade --install api ./chart -n app
helm rollback api 3 -n app
```

**Коротко:**

- Helm спрощує керування складними deployment-наборами.
- Дає параметризацію через values і контроль версій релізів.
- Широко використовується для platform і app deployment.

</details>

<details>
<summary>108. Що таке Helm Chart?</summary>

#### Kubernetes

Helm Chart — пакунок шаблонів Kubernetes-ресурсів і metadata для інсталяції
застосунку.

Типова структура:

- `Chart.yaml`
- `values.yaml`
- `templates/`

Chart може мати залежності (`dependencies`).

**Коротко:**

- Chart = інсталяційний шаблон застосунку для Helm.
- Містить шаблони, дефолтні values і метадані.
- Дозволяє повторно використовувати deployment-патерни.

</details>

<details>
<summary>109. Різниця між Helm і Kustomize?</summary>

#### Kubernetes

- Helm: шаблонізація (Go templates), values, release management.
- Kustomize: patch/overlay без шаблонізатора, ближче до "чистого YAML".

Helm зручніший для reusable пакетів; Kustomize — для керованих overlays
усередині одного репо/платформи.

**Коротко:**

- Helm = пакетування + шаблонізація + релізи.
- Kustomize = композиція і патчинг маніфестів.
- Обирають за моделлю керування, а не за "краще/гірше".

</details>

<details>
<summary>110. Які best practices використання Helm?</summary>

#### Kubernetes

Практики для production:

- тримати values мінімальними і явними по середовищах;
- уникати надмірної логіки в templates;
- додавати `values.schema.json` для валідації;
- фіксувати версії chart і dependencies;
- перевіряти рендер (`helm template`, `helm lint`) у CI.

**Коротко:**

- Helm має залишатися читабельним і передбачуваним.
- Валідація values і CI-перевірки критичні.
- Версійний контроль chart/dependencies знижує регресії.

</details>

<details>
<summary>111. Що таке Kubernetes Operator?</summary>

#### Kubernetes

Operator — це контролер, який автоматизує доменну операційну логіку для
складного застосунку через CRD + reconciliation.

Типові задачі Operator:

- bootstrap кластера сервісу;
- backup/restore;
- failover;
- upgrade orchestration.

**Коротко:**

- Operator кодує runbook у вигляді controller logic.
- Використовується для складних stateful систем.
- Працює як розширення control plane.

</details>

<details>
<summary>112. Що таке Custom Resource (CR)?</summary>

#### Kubernetes

Custom Resource (CR) — конкретний обʼєкт користувацького API-типу, визначеного
через CRD.

Приклад: `PostgresCluster/my-db` як інстанс власного ресурсу.

CR описує desired state, який обробляє відповідний controller/operator.

**Коротко:**

- CR — це екземпляр кастомного ресурсу.
- Дозволяє керувати доменною логікою через Kubernetes API.
- Сам по собі CR не працює без контролера, який його reconciles.

</details>

<details>
<summary>113. Що таке Custom Resource Definition (CRD)?</summary>

#### Kubernetes

CRD — ресурс, який розширює Kubernetes API новим типом обʼєкта.

CRD визначає:

- групу/версію/вид ресурсу;
- схему (OpenAPI validation);
- scope (Namespaced або Cluster).

Після створення CRD у кластері зʼявляються нові API endpoints.

**Коротко:**

- CRD додає нові типи ресурсів у Kubernetes.
- Дає формальний контракт і валідацію для custom API.
- Є фундаментом для Operators.

</details>

<details>
<summary>114. Коли варто створювати Operator?</summary>

#### Kubernetes

Operator варто створювати, коли ручний runbook для сервісу складний,
повторюваний і критичний для надійності.

Сигнали, що Operator виправданий:

- складні day-2 операції (backup, failover, resharding, upgrade);
- багато однакових інсталяцій сервісу;
- високий ризик помилок при ручному керуванні;
- потреба у self-healing на рівні доменної логіки.

**Коротко:**

- Operator потрібен для автоматизації складної operational логіки.
- Для простих stateless сервісів зазвичай достатньо Deployment/Helm.
- Вартість розробки Operator має окупатися стабільністю і масштабом.

</details>

<details>
<summary>115. Як забезпечити High Availability кластера?</summary>

#### Kubernetes

High Availability (HA) кластера забезпечується на рівні control plane, worker
nodes і застосунків.

Практичний baseline:

- мінімум 3 control plane вузли (etcd quorum);
- розподіл control plane та node pools по різних AZ;
- кілька реплік критичних сервісів + anti-affinity;
- `PodDisruptionBudget` для захисту доступності;
- зовнішній/керований load balancer перед API server.

**Коротко:**

- HA = відсутність single point of failure на всіх рівнях.
- Критичний мінімум: 3 control plane і зональна відмовостійкість.
- Для сервісів потрібні репліки, anti-affinity і PDB.

</details>

<details>
<summary>116. Як виконати backup та restore etcd?</summary>

#### Kubernetes

Backup роблять через snapshot `etcdctl`, restore — із цього snapshot у новий або
відновлений etcd data dir.

Типовий потік:

```bash
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%F).db
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-2026-02-26.db -w table
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-2026-02-26.db --data-dir /var/lib/etcd-restore
```

Практика production:

- регулярний автоматизований snapshot schedule;
- зберігання копій поза кластером;
- регулярний тест restore (DR drill).

**Коротко:**

- etcd backup/restore — ключова процедура disaster recovery.
- Snapshot без перевіреного restore-процесу недостатній.
- Backup має бути регулярним і протестованим.

</details>

<details>
<summary>117. Що станеться, якщо etcd стане недоступним?</summary>

#### Kubernetes

Якщо etcd недоступний, control plane втрачає доступ до джерела істини стану
кластера.

Наслідки:

- нові create/update/delete операції через API зазвичай не виконуються;
- controllers/scheduler не можуть коректно reconcile новий стан;
- вже запущені Pod на worker nodes можуть продовжити працювати певний час.

Фактично кластер переходить у режим деградації керованості.

**Коротко:**

- Недоступний etcd блокує керування кластером.
- Поточний workload може жити, але керованість різко падає.
- Тому etcd HA і restore-план критично важливі.

</details>

<details>
<summary>118. Як розподіляти навантаження між зонами доступності?</summary>

#### Kubernetes

Для multi-AZ розподілу використовують scheduling-політики та zonal інфраструктуру.

Базові механізми:

- node groups у кількох AZ;
- `topologySpreadConstraints` для рівномірного розміщення Pod;
- pod anti-affinity по zone/hostname;
- zonal-aware storage і load balancer;
- перевірка cross-zone latency/cost.

**Коротко:**

- Розподіл по AZ підвищує стійкість до зональних збоїв.
- `topologySpreadConstraints` і anti-affinity — ключові інструменти.
- Потрібно узгоджувати compute, network і storage по зонах.

</details>

<details>
<summary>119. Як працюють readiness і liveness probes?</summary>

#### Kubernetes

`readinessProbe` визначає, чи Pod готовий приймати трафік.
`livenessProbe` визначає, чи контейнер "живий" і чи треба його перезапустити.

Поведінка:

- readiness fail -> Pod прибирається з Service endpoints;
- liveness fail -> kubelet перезапускає контейнер.

Важливо правильно налаштувати `initialDelaySeconds`, `timeoutSeconds`,
`failureThreshold`, щоб уникати хибних перезапусків.

**Коротко:**

- Readiness керує маршрутизацією трафіку.
- Liveness керує автовідновленням завислих контейнерів.
- Неправильні probe-параметри часто викликають нестабільність.

</details>

<details>
<summary>120. Що таке startup probe і коли його використовувати?</summary>

#### Kubernetes

`startupProbe` використовується для повільного старту контейнерів і тимчасово
відключає перевірки liveness/readiness, доки застосунок не підніметься.

Коли потрібен:

- довгий bootstrap (міграції, кеш-прогрів, великі JVM/.NET старти);
- liveness занадто рано вбиває контейнер під час ініціалізації.

Після успішного startup probe починають діяти readiness/liveness.

**Коротко:**

- Startup probe захищає повільний старт від помилкових рестартів.
- Використовується для "важких" застосунків із довгою ініціалізацією.
- Після завершення startup працюють звичайні probes.

</details>
