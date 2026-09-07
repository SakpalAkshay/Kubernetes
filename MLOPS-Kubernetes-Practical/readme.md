# Build Your First Kubernetes Project (MLOps) — Part 2 Notes

Hands-on Hinglish notes: ek Flask app ko Docker se containerize karke Minikube ke Kubernetes cluster pe deploy karna, self-healing aur load balancing test karna, aur ye samajhna ki real production (on-prem / cloud) me yahi cheez kaise hoti hai.

> Ye **Part 2** hai. Part 1 me Distributed Computing, Monolithic vs Microservices aur Kubernetes Internals (Control Plane, Pods, Kubelet, etcd, Service, Namespace, ReplicaSet) theory me cover ho chuke hain. Agar wo concepts comfortable nahi hain to pehle Part 1 dekh lo — ye video usi ke upar bani hai.

---

## Table of Contents

- [System Requirements (RAM ki Baat)](#system-requirements-ram-ki-baat)
- [Project ka Poora Plan](#project-ka-poora-plan)
- [Step 1: GitHub Repo Banao aur Clone Karo](#step-1-github-repo-banao-aur-clone-karo)
- [Step 2: App Banao aur Locally Test Karo](#step-2-app-banao-aur-locally-test-karo)
  - [Itna Simple App Kyun, ML App Kyun Nahi?](#itna-simple-app-kyun-ml-app-kyun-nahi)
- [Step 3: Docker Hub aur Docker Desktop Ready Rakho](#step-3-docker-hub-aur-docker-desktop-ready-rakho)
- [Step 4: Dockerfile Banao](#step-4-dockerfile-banao)
- [Step 5: Docker Image Build aur Run Karo](#step-5-docker-image-build-aur-run-karo)
- [Step 6: Minikube — Cluster Banane ka Tool](#step-6-minikube--cluster-banane-ka-tool)
  - [Minikube Install Karna](#minikube-install-karna)
  - [Cluster Start Karna](#cluster-start-karna)
  - [Agar minikube start pe Error Aaye — Troubleshooting](#agar-minikube-start-pe-error-aaye--troubleshooting)
  - [Minikube Production Grade Nahi Hai](#minikube-production-grade-nahi-hai)
- [Step 7: Cluster ko Explore Karo (kubectl)](#step-7-cluster-ko-explore-karo-kubectl)
  - [minikube Commands vs kubectl Commands](#minikube-commands-vs-kubectl-commands)
  - [Pods Dekhna](#pods-dekhna)
  - [Nodes Dekhna aur Extra Worker Node Banana](#nodes-dekhna-aur-extra-worker-node-banana)
- [Step 8: Docker Image ko Cluster ke Andar Bhejo](#step-8-docker-image-ko-cluster-ke-andar-bhejo)
- [Step 9: deployment.yaml — Kube Manifest Samjho](#step-9-deploymentyaml--kube-manifest-samjho)
  - [YAML Likhne ka Shortcut (VS Code Extension)](#yaml-likhne-ka-shortcut-vs-code-extension)
  - [Deployment Section Line by Line](#deployment-section-line-by-line)
  - [Service Section Line by Line](#service-section-line-by-line)
- [Step 10: Deploy Karo](#step-10-deploy-karo)
- [Step 11: Self-Healing ka Demo — Pod Delete Karke Dekho](#step-11-self-healing-ka-demo--pod-delete-karke-dekho)
- [Step 12: Service ke Through App Access Karo](#step-12-service-ke-through-app-access-karo)
  - [Live Debugging: ImagePullBackOff Error](#live-debugging-imagepullbackoff-error)
- [Minikube Dashboard](#minikube-dashboard)
- [Endpoints, Services, Ingress aur Egress](#endpoints-services-ingress-aur-egress)
- [Load Balancing ka Demo (Postman se)](#load-balancing-ka-demo-postman-se)
- [Cluster Band Karna](#cluster-band-karna)
- [Image ko Docker Hub / ECR se Pull Karna](#image-ko-docker-hub--ecr-se-pull-karna)
- [Kubernetes ko CI/CD Flow me Kaise Fit Karein](#kubernetes-ko-cicd-flow-me-kaise-fit-karein)
- [Real Life me Cluster Kahan Banta Hai](#real-life-me-cluster-kahan-banta-hai)
  - [On-Premise Servers](#on-premise-servers)
  - [Managed Kubernetes Services (EKS / AKS / GKE)](#managed-kubernetes-services-eks--aks--gke)
- [Summary / Key Takeaways](#summary--key-takeaways)

---

## System Requirements (RAM ki Baat)

Ye project locally chalane wala hai, to RAM matter karti hai:

- Instructor ke system pe **24 GB RAM** hai (pehle 8 GB tha, isi kaam ke liye upgrade karana pada — pichhli recording me system crash ho gaya tha).
- **12–16 GB** ho to bilkul aaram se chal jayega.
- **8 GB** me bhi ho jayega — kyunki aap screen recording nahi kar rahe, jo RAM utilization kaafi badha deti hai. Instructor ne 8 GB pe ye poora kaam successfully kiya tha, bas recording ke bina.
- **32 GB ya usse zyada** hai to aap simple Flask app ki jagah ek proper **Machine Learning application** use kar sakte ho.

> **Sabse zaroori advice:** RAM kitni hai, laptop hai bhi ya nahi — pehle poora video ek baar dekh lo aur ek working understanding banao. Jab lage ki cheezein samajh aa rahi hain, tab doosri baar dekhte hue parallel me practically karo.

RAM check karne ke liye Task Manager (`Ctrl + Shift + Esc`) → **Performance** → **Memory**.

---

## Project ka Poora Plan

Steps ye rahenge:

1. App build karo aur locally test karo
2. Docker Hub aur Docker Desktop dono chalu rakho, dono pe sign in raho
3. Project me **Dockerfile** banao
4. Docker image build karo aur test karo
5. **deployment.yaml** (Kube Manifest) banao
6. **Minikube** install karo aur cluster start karo
7. Cluster explore karo (`kubectl` commands)
8. Image ko cluster ke andar load karo
9. Deploy karo aur service ke through access karo
10. Self-healing aur load balancing test karo
11. Samjho ki ye sab production (cloud / on-prem) me kaise hota hai

---

## Step 1: GitHub Repo Banao aur Clone Karo

Sabse pehle ek naya GitHub repo banao — naam `k8s-kubernetes-mini-project` type ka kuch.

> **Ek chhota sa fact:** Kubernetes ko professionals **K8s** bolte hain, kyunki shuruat ke **K** aur ant ke **s** ke beech me exactly **8 letters** hote hain.

Repo banate waqt:

- **Public** rakho (taaki code aur notes sabko mile)
- README file add karo
- Python ka `.gitignore` add karo
- License add kar do

Phir apne local pe clone karo — folder me `Shift + Right Click` karke PowerShell kholo:

```bash
git clone <your-repo-url>
cd <repo-folder>
code .
```

Ab khaali project directory VS Code me ready hai.

---

## Step 2: App Banao aur Locally Test Karo

Ek bahut simple **Flask** app banaya gaya hai — kuch hi lines ka code:

- Ek UI provide karta hai jo aapka **naam input** me leta hai
- Aur message print karta hai: *"Hello \<your name\>, welcome to the Kubernetes Test Application"*

Beautify karne ke liye ek `static/` folder me CSS aur `templates/` folder me HTML rakha gaya hai.

> **Ek honest tip instructor se:** "Mujhe HTML/CSS develop karna nahi aata — padh ke interpret kar sakta hoon bas. Toh maine apna Flask app LLM ko diya aur usse corresponding HTML/CSS bana liya." Aap bhi yahi kar sakte ho.

Locally chalao:

```bash
python app.py
```

Ye **port 5000** pe run hoga. Browser me kholo:

```
http://localhost:5000
```

Naam enter karo, Submit dabao, aur message dikh jayega. App ready hai.

### Itna Simple App Kyun, ML App Kyun Nahi?

Ye sawal aana natural hai. Ideally ek MLOps Engineer Kubernetes pe **Machine Learning application** hi deploy karega. Lekin:

- ML app is machine ke liye bahut **heavy** ho jata — pichhli recording me system crash ho chuka tha
- Isliye ek **lightweight application** ka idea liya gaya

**Aapke liye suggestion:** Chahe aapke paas 32/64 GB RAM ho, **pehle is lightweight app ke saath hi poora flow karo**. Jab confidence aa jaye ki cheezein samajh me aa rahi hain, tab agle trial me ML application utha lena aur usko is app ki jagah replace kar dena.

Is point pe aapka **poora focus sirf ek baat pe hona chahiye**: application chahe kuch bhi ho (ML ho ya Flask), usko **Kubernetes me deploy karte kaise hain**.

---

## Step 3: Docker Hub aur Docker Desktop Ready Rakho

- **Docker Desktop** apne system pe chalu karo
- **Docker Hub** pe bhi sign in raho
- Dono pe **already signed-in** hona chahiye — sign-in ka option nahi dikhna chahiye

Docker Desktop me existing test images/containers ko clean kar do agar scratch se karna hai — pehle **Containers** se delete karo, phir **Images** se.

---

## Step 4: Dockerfile Banao

Project root me `Dockerfile` chahiye. Isme jo cheezein hain:

- **Base image**: `python:3.9-slim` — slim isliye taaki image lightweight rahe
- **Do environment variables** jo Python ko `.pyc` files banane se rokte hain — isse image/container lightweight rehta hai aur system pe kam stress padta hai

  > Ye trick instructor ko pehle nahi pata thi — LLM se search karke nikali, kyunki image ko jitna ho sake lightweight rakhna tha.

- Ek **working directory** image ke andar create karna
- `requirements.txt`, `app.py`, aur HTML/CSS files ko us directory me copy karna
- Dependencies install karna (Flask aur uska version)
- App run karne ka command

> Dockerfile ke har instruction pe detail discussion is playlist ke **Docker tutorial** me ho chuki hai. Agar yahan atak rahe ho to pehle wo dekh lo.

---

## Step 5: Docker Image Build aur Run Karo

Pehle check karo ki abhi kya images hain:

```bash
docker images
```

Ya Docker Desktop → **Images** tab. Abhi sirf Minikube ka image dikhega (agar Minikube install hai), aapka app nahi.

**Image build karo:**

```bash
docker build -t kubernetes-test-app:latest .
```

- `-t` ek flag hai **tag** dene ke liye
- Colon (`:`) se pehle **app ka naam**, baad me **tag** — ye `latest`, `v1`, `v2` kuch bhi ho sakta hai

Pehli baar build me thoda time lagega. Build ke baad verify karo:

```bash
docker images
```

Ab `kubernetes-test-app` dikhega, size approx **130 MB**.

> **Dhyan rakho:** Ye ek lightweight app hai phir bhi 130 MB ka hai. Agar aap ek **Machine Learning application** use karoge to uska size roughly **700 MB se 1–1.25 GB** tak ja sakta hai.

**Image ko run karke test karo:**

```bash
docker run -p 5000:5000 kubernetes-test-app:latest
```

- `-p` **port mapping** ka flag hai
- Image ke andar 5000 expose hai, aur hum apne system ke 5000 pe usko map kar rahe hain

Browser me `localhost:5000` kholo — ab jo app chal raha hai wo **Docker image se execute ho raha hai**, direct Python se nahi. `Ctrl + C` karoge to service band ho jayegi aur page "not reachable" ho jayega.

Docker Desktop → **Containers** me ek randomly named container (jaise "crazy-yeti") dikhega — Docker Desktop khud ye naam generate karta hai. Wahin se aap use start/stop bhi kar sakte ho.

---

## Step 6: Minikube — Cluster Banane ka Tool

Part 1 me humne cluster, control plane, worker nodes sab padha — lekin **ek cheez discuss nahi ki thi: ye cluster banta kaise hai?**

Cluster chahe aapke local machine pe ho, on-premise server pe ho, ya AWS EC2 jaise server pe — **wo by default nahi hota, usko banana padta hai**.

**Minikube** apne local machine pe cluster set up karne ka sabse aasan tareeka hai. Minikube khud bhi ek service hai jo **Docker ke upar hi chalti hai**.

### Minikube Install Karna

1. Minikube ke official start page pe jao (`minikube.sigs.k8s.io/docs/start`)
2. Apne **operating system** ke hisaab se download option select karo
3. Windows pe: `.exe` download option pe click karo — aapko ek **PowerShell command** milega
4. **PowerShell ko "Run as Administrator"** se kholo (ye zaroori hai, page pe bhi likha hota hai)
5. Pehla command chalao — ye Minikube **download** karega
6. Doosra command chalao — ye Minikube ko **configure** karega

> **Sabse important:** Ye dono commands chalane ke baad **apna poora system restart karo**. Warna `minikube start` pe error aayega.

> Agar ye link future me kaam na kare (6 mahine ya saal-do-saal baad), to bas official docs pe apne OS ke hisaab se latest instructions le lena.

### Cluster Start Karna

```bash
minikube start
```

Bas — ek simple command se aapka cluster ban jayega. Agar output ke ant me ye message dikhe:

> *"kubectl is now configured to use the cluster and default namespace"*

...to samajh lo **aapka cluster successfully create ho chuka hai**.

Status check karo:

```bash
minikube status
```

Ye batayega ki aapka **control plane, host, kubelet, apiserver** — sab running hain.

### Agar minikube start pe Error Aaye — Troubleshooting

Bahut logon ko yahan ek lamba, gandha sa error aata hai (`Failing to connect to...` aur proxy use karne ka suggestion, saath me lambe error logs). Ye Minikube ka temporary issue/bug bhi ho sakta hai. Instructor ne StackOverflow, official docs aur LLM — sab jagah search kiya. Fix ye steps hain:

**1. Internet connection check karo**

Minikube internally apni repository se kaafi kuch download karta hai — **offline ye kaam nahi hoga**.

**2. Proxy ka issue dekho**

*Agar aap proxy use karte ho*, to Minikube ko bhi proxy batao:

```bash
minikube start --docker-env HTTP_PROXY=<your-proxy> --docker-env HTTPS_PROXY=<your-proxy>
```

*Agar aapko pata bhi nahi ki proxy kya hota hai*, tension mat lo — bas notes me diye gaye **teen commands** ek-ek karke PowerShell pe chala do, jo system pe kisi bhi type ka proxy set na hone ki guarantee dete hain.

**3. Registry connectivity check karo**

Minikube internet pe ek Kubernetes registry ko hit karta hai. `nslookup` se check karo ki us registry se connection ban raha hai ya nahi:

```bash
nslookup <kubernetes-registry-host>
```

Agar error ki jagah normal logs dikhein — **your connection is set**.

**4. Purana failed instance clean karo**

Error wale attempt se bhi Minikube ka ek instance chalu ho jata hai. Usko hatao:

```bash
minikube stop
minikube delete --all
```

Iske baad **system restart karo**, phir dobara `minikube start` chalao. **99% cases me error resolve ho jayega.**

Agar phir bhi na ho — yahi wo jagah hai jahan aapko apni **debugging skills** brush up karni hain. Logs uthao, internet pe search karo.

### Minikube Production Grade Nahi Hai

Ye clearly samajh lo: **Minikube sirf experimental projects, learning, development aur education purpose ke liye achha hai.** Aisa kabhi nahi hoga ki aap kisi employer ya client ke industry-grade project me Kubernetes deployment kar rahe ho aur wahan Minikube use karo.

Production setups (on-prem / cloud) pe baad me detail me baat hai.

---

## Step 7: Cluster ko Explore Karo (kubectl)

### minikube Commands vs kubectl Commands

Do type ke commands honge:

- **`minikube ...`** — jaise `minikube status`, `minikube dashboard`, `minikube stop`, `minikube delete`. Ye aapki **professional life me kaam nahi aayenge**, sirf local learning ke liye hain.
- **`kubectl ...`** — ye **har jagah same** kaam karenge. Chahe aapka organization Kubernetes on-premise physical server pe chala raha ho ya cloud setup pe — **ye saare kubectl commands wahan bhi as-it-is use honge.**

**Sab kuch ek saath dekhne ka command:**

```bash
kubectl get all -A
```

Ye ek hi baar me namespaces se lekar andar chal rahe saare elements (API server, controller, etc.) dikha deta hai — thoda overwhelming lagta hai, isliye hum ise **block by block** dekhenge.

### Pods Dekhna

```bash
kubectl get pods -A
```

Ye aapke cluster ke saare pods dikhata hai. Ab yahan ek **bahut important realization** hai:

Aapne abhi tak apna app cluster me daala hi nahi, phir bhi itne saare pods kyun dikh rahe hain? Kyunki Part 1 me jo components humne padhe the — **etcd, API Server, kube-controller, scheduler, kube-proxy** — **ye sab khud microservices hain jo chhote-chhote Docker images me cluster ke andar chal rahe hain.**

> Isiliye internet pe kahin bhi Docker search karo ya Kubernetes — dono ka naam saath aata hai: **Kubernetes khud chhoti-chhoti Docker services ke upar run karta hai.**

### Nodes Dekhna aur Extra Worker Node Banana

```bash
kubectl get nodes -A
```

Abhi sirf **ek node** dikhega — **control plane**. Kyunki cluster banate waqt extra worker nodes explicitly maange hi nahi gaye the.

**Sirf ek node kyun kaafi hai?**

1. Ye zaroori nahi ki app kisi alag worker node me hi deploy ho — **control plane ke andar bhi deployment possible hai**. Agar aap explicitly naye worker node me deploy karna chahte ho, tab aap mention kar sakte ho.
2. Har node aapke **physical device se resources leta hai** — RAM, CPU, memory. Storage shayad problem na ho, lekin **CPU dikkat de sakta hai**. Recording ke dauraan ye aur bhi heavy pad jata hai.

**Agar aapko zyada nodes chahiye** (8–12 GB RAM hai to ek extra node aaram se ban jayega):

```bash
minikube start --nodes=2
```

Ab `kubectl get nodes -A` chalane pe aapko dono (ya jitne banaye) nodes dikh jayenge.

---

## Step 8: Docker Image ko Cluster ke Andar Bhejo

Pehle dekho ki cluster ke andar kaun-kaun se images hain:

```bash
minikube image list
```

Yahan scheduler, kube-proxy, controller-manager sab ke images dikhenge — **lekin aapka `kubernetes-test-app` nahi hoga**. Isliye pods me bhi wo nahi chal raha.

**Image cluster me load karo:**

```bash
minikube image load kubernetes-test-app:latest
```

Ek minute lag sakta hai. Phir dobara verify karo:

```bash
minikube image list
```

Ab list ke aakhir me aapka **`kubernetes-test-app`** dikhega — image successfully cluster ke andar aa chuki hai.

> **Ek important note:** Real CI/CD flow me image **directly deployment me nahi jaati** — wo pehle ek **repository** me jaati hai. Ye ek public repo (Docker Hub) ya cloud repo (AWS ECR) ho sakta hai. Hum abhi direct push kar rahe hain, lekin Docker Hub / ECR wala tareeka bhi aage cover hai.

---

## Step 9: deployment.yaml — Kube Manifest Samjho

Ye wahi **Kube Manifest** file hai jiske baare me Part 1 me baat hui thi. Isi ke andar hum instructions likhte hain ki kaunsi image ko Kubernetes me kaise deploy karna hai.

### YAML Likhne ka Shortcut (VS Code Extension)

Sawal aata hai: "Itni badi YAML file likhenge kaise?"

Aaj ki date me asli challenge YAML likhna nahi, **logic samajhna** hai — ki ye ek **key** hai (`kind`) aur ye uski **value** (`Deployment`). Agar ye nested structure logically samajh aata hai, to likhna challenge nahi.

**Shortcut:**

1. VS Code → **Extensions** → search karo **Kubernetes**
2. Sabse upar **Microsoft** ka Kubernetes extension install/enable karo
3. Ab kisi bhi `.yaml` file me sirf `deployment` type karke **Tab** dabao — poora deployment skeleton generate ho jayega
4. Isi tarah `service` type karke Tab dabao — service skeleton bhi bann jayega

> Ek YAML file me **do configs** ek saath rakh sakte ho — beech me **teen hyphen (`---`)** separator ka kaam karta hai. **Teen se zyada hyphen mat daalna, error ho jayega.**

Bahut log deployment aur service ki alag-alag files banate hain, lekin ek hi file me dono rakhna bhi bilkul theek hai.

### Deployment Section Line by Line

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-test-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: kubernetes-test-app
  template:
    metadata:
      labels:
        app: kubernetes-test-app
    spec:
      containers:
        - name: kubernetes-test-app
          image: kubernetes-test-app:latest
          imagePullPolicy: Never
          # image: <your-dockerhub-username>/<repo>:<tag>
          resources:
            limits:
              memory: "64Mi"
              cpu: "200m"
          ports:
            - containerPort: 5000
```

**Har field ka matlab:**

- **`apiVersion`** — version, jo bhi aap dena chahein.
- **`kind: Deployment`** — **ismein chhedchhad mat karna.** Ye `Deployment` hi rehna chahiye, kyunki ye ek Kube Manifest file hai aur Kubernetes isi information ko internally use karta hai.
- **`name`** — aapke application ka naam. Jo naam aap rakhoge, **jahan-jahan wo naam aata hai har jagah wahi rakhna hoga.** Pehli baar kar rahe ho to same naam rakhna better hai.
- **`replicas: 2`** — "mujhe minimum **2 pods** chahiye." Duniya idhar se udhar ho jaye, Kubernetes 2 replicas up-and-running rakhega hi. Zyada scale-up ho sakta hai, lekin minimum 2 rahenge.
- **`image`** — app ka naam + colon + **tag**. Build karte waqt jo tag diya tha (`latest`, `v1`, `v3`) wahi yahan dena hai. Kal ko naya version banaya to bas `latest` ki jagah `v3` likh dena — as simple as that.
- **`imagePullPolicy: Never`** — **ye part bahut important hai.** By default Kubernetes aapki image ko ek **remote repo me dhoondhta hai** — usko kya pata ki aap locally apne system se bhej rahe ho. Agar aap nahi chahte ki wo remote dhoondhe, to `imagePullPolicy` ko `Never` set kar do — tab wo image locally hi utha lega.
- **Commented-out image line** — agar aap **local se image push nahi karna chahte**, to `imagePullPolicy: Never` wali line comment out kar do aur uski jagah apne repo ka path use karo (`<username>/<repo>:<tag>`). Ye path aapko ECR, Docker Hub, ya aapke employer se mil jayega.
- **`resources.limits`** — node ko kitna resource dena hai. `memory: 64Mi` aur `cpu: 200m` (200 milicore). **200 milicore ka matlab:** agar aapke paas 2-core CPU hai, to ek core ka 1/5th. Bas rule define karna tha — **baaki Kubernetes khud sambhal lega.**
- **`containerPort: 5000`** — is port pe app deploy hoga.

> ⚠️ **Yahi wo galti hai jo video me live hui:** `imagePullPolicy: Never` comment out reh gaya tha, jiski wajah se Kubernetes image ko cluster ke bahar dhoondhne laga. Rule simple hai — **ya to `imagePullPolicy: Never` + local image use karo, ya dono comment out karke remote repo wali line use karo.** Beech ka koi option nahi.

### Service Section Line by Line

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-test-app
spec:
  type: LoadBalancer
  selector:
    app: kubernetes-test-app
  ports:
    - port: 8080
      targetPort: 5000
```

- **`kind: Service`** — isse Kubernetes samajh jata hai ki **ye service config hai**, deployment nahi.
- **`port: 8080`** — "mera deployment bhale hi 5000 pe ho, but aap use mere liye 8080 pe available karao."
- **`targetPort: 5000`** — port mapping, jo upar deployment ke `containerPort` se aa raha hai.

**Service ka refresher (Part 1 se):**

Jab hum EC2 pe app deploy karte the, to har instance ke **inbound rules / security credentials** me jaake manually port allow karna padta tha (jaise 8080 public access ke liye). Ab socho — aap **kitne nodes banaoge aur kitni baar ye changes karoge?**

Service file kehti hai: *"in nodes ya pods ka endpoint jo bhi ho, main aapko **ek common endpoint** de dungi. Aap usko hit karo, aur ye chinta mujhpe chhod do ki service kaunse node ya pod se aa rahi hai."*

---

## Step 10: Deploy Karo

**Zaroori:** ye command **project directory ke andar se** chalana hai.

```bash
kubectl apply -f deployment.yaml
```

Do messages aane chahiye — deployment created aur service created.

**Deployment delete karne ke liye:**

```bash
kubectl delete -f deployment.yaml
```

> **Yaad rakho:** isse **deployment delete hoti hai, image nahi** — image cluster ke andar hi reh jaati hai. Image delete karne ka command alag hota hai (abhi utna essential nahi).

Ab pods check karo:

```bash
kubectl get pods -A
```

Pehle se maujood Kubernetes internal pods (etcd, API server, etc.) ke **upar do naye records** dikhenge — **aapka `kubernetes-test-app` do baar chal raha hai.**

**Do baar kyun?** Kyunki `deployment.yaml` me humne explicitly `replicas: 2` maanga tha.

> Nodes phir bhi ek hi rahega (control plane), kyunki hum single node pe kaam kar rahe hain.

---

## Step 11: Self-Healing ka Demo — Pod Delete Karke Dekho

Ek pod manually delete karke dekhte hain ki Kubernetes kya karta hai:

```bash
kubectl get pods
kubectl delete pod <pod-name>
```

Delete confirm ho jayega. Ab dobara check karo:

```bash
kubectl get pods
```

**Result:** purana pod gayab hai, lekin uski jagah **turant ek naya pod spin off ho chuka hai** — age dikhega **~15 seconds**.

Ab khud socho: kya aap **15 second me** ek EC2 instance create karke, usme deploy karke, service up-and-running karke, security credentials update karke apni service start kar paate?

> **This is the power that you get on top of Kubernetes.** Do pods maange hain to duniya idhar se udhar ho jaye — do pods rahenge hi.

Real life me ye failure kisi bhi wajah se ho sakta hai — internal system issue, crash, ya jis area me server hai wahan koi disaster. **Kubernetes khud handle kar leta hai.**

> ⚠️ **Note:** Aap real life me kabhi manually pods delete nahi karte. Ye sirf **demo** ke liye tha.

---

## Step 12: Service ke Through App Access Karo

App cluster ke andar 2 pods me deploy to ho gaya, lekin **use bhi to karna hai**. Aap kaunse pod ko hit karoge? — **Isi liye service file likhi thi.**

```bash
minikube service kubernetes-test-app
```

Ye khud hi browser me app launch kar dega, ek URL pe (jaise `localhost:64806`).

> ⚠️ **Bahut important:** Har baar jab aap Minikube service restart karoge, **ye port change ho jayega.** Aaj 64806 hai, kal kuch aur hoga.

### Live Debugging: ImagePullBackOff Error

Video me yahan live ek error aaya, jo seekhne layak hai:

```bash
kubectl get pods
```

Status aaya: **`ImagePullBackOff`**

**Kya galti thi?** `deployment.yaml` me `imagePullPolicy: Never` **comment out** reh gaya tha. Iska matlab Kubernetes image ko **cluster ke andar dhoondhne ki jagah bahar** (remote repo me) dhoondh raha tha — aur wahan wo thi hi nahi.

**Fix:**

```bash
kubectl delete -f deployment.yaml
# deployment.yaml me imagePullPolicy: Never uncomment karo
kubectl apply -f deployment.yaml
kubectl get pods
```

Ab status **`Running`** aa gaya, aur service properly serve karne lagi.

> Instructor ka honest takeaway: *"Main definitely aap me se zyadatar logon se thoda zyada experienced hoon, phir bhi mujhe is type ke problems ho jaate hain. To aap bhi expect karo ki aapke kaam me bhi is type ki dikkatein aayengi."* Debug karna hi asli skill hai.

---

## Minikube Dashboard

Minikube ek monitoring dashboard bhi deta hai. **Ek naye PowerShell instance me** chalao (taaki service chalti rahe):

```bash
minikube dashboard
```

Ye ek naye port pe dashboard bana deta hai jahan aap dekh sakte ho:

- **Deployments** — kitne hain
- **Pods** — 2 running (green = 100% healthy)
- **ReplicaSets** — 1
- Har cheez ka **health check**, chart form me

Abhi ye thoda monotone dikhega kyunki humara deployment simple hai. Multiple services deploy hone pe yahan proper charts dikhte hain ki kitne up hain aur kitne down. Kahin bhi **red/blue** dikhe to samajh jao ki wahan issue hai.

> **Real life me ye use nahi hota.** Ye Minikube ka ek additional feature hai taaki aap monitoring/dashboarding ke concept se familiar ho jao. Production me iske liye **Prometheus** (logs, tracking, monitoring) aur **Grafana** (dashboards) use hote hain.

---

## Endpoints, Services, Ingress aur Egress

```bash
kubectl get endpoints
kubectl get services
```

`get endpoints` aapko **control plane ka aur dono pods ka endpoint** dikhata hai — jo unke **internal communication** ke liye chahiye hota hai, aur jo bahar exposed hain wo bhi.

> Ek MLOps Engineer ke liye ye abhi utna kaam ka nahi hai, lekin ek **Kubernetes Administrator** ke liye ye details bahut zaroori hoti hain kyunki wo in cheezon pe heavily kaam karte hain.

**Ingress vs Egress:**

Cluster pe do interfaces hote hain:

- **Ingress (I)** — agar aap cluster ke **andar** koi information, communication ya data inject karna chahte ho, to uska **gateway = Ingress**
- **Egress (E)** — cluster se kuch information **bahar** bhejni ho (alerting ke liye, ya koi output jo bahar communicate karna hai), to uska **gateway = Egress**

> Abhi humare level pe inki koi zaroorat nahi hai, lekin **in do terms se aware zaroor raho** — aage chalke sunoge to blank nahi hone chahiye.

---

## Load Balancing ka Demo (Postman se)

Part 1 me theory thi ki Kubernetes user requests ko har pod ki capacity ke hisaab se distribute karta hai, aur aapko rules define nahi karne padte. Ab practically dekhte hain.

**Setup:** Do PowerShell windows side-by-side kholo aur dono pods ke **live logs** dekho:

```bash
kubectl get pods
kubectl logs -f <pod-1-name>
```

```bash
kubectl logs -f <pod-2-name>
```

> `-f` flag (follow) zaroori hai — warna logs turant end ho jaate hain. Ye galti video me bhi hui thi.

**Manual (gareeb) tareeka:** Browser me app kholo, baar-baar naam submit karo, aur dekho ki `200` wala message kabhi left window me jata hai kabhi right me. Load balancing ho rahi hai — lekin ye testing ka tareeka nahi hai.

**Industry-grade tareeka: Postman**

Postman download aur install karo (`postman download` search karke pehle link se).

**Request set up karo:**

1. **New** → **HTTP**
2. Method: **POST** (kyunki hum naam post kar rahe hain)
3. URL: wahi URL jahan `minikube service` ne aapka app expose kiya hai
4. **Body** → **form-data** → key-value add karo:
   - **Key**: `name` (yahi key aapke `app.py` me use hui hai)
   - **Value**: `Postman User`
5. **Save** karo:
   - **Request name**: `k8s-test`
   - **Collection name**: `Kubernetes Mini Project`

> **Collection vs Request ka concept:** Collection basically aapke **project ka naam** hota hai (`Fraud Detection`, `Classification`). Uske andar chhote-chhote services ko hit karne ke APIs hote hain — ek service jo MongoDB se data fetch karti hai, ek jo model training start karti hai, ek prediction ke liye. Ye saare services usi collection ke andar aate hain.

**Send** karo — `200 OK` aana chahiye, aur response me (Preview me) *"Hello Postman User, welcome to the Kubernetes Test Application"* dikhega. **Matlab app seedha cluster ke andar se respond kar raha hai.**

**Ab load bhejte hain:**

1. Postman me **Collections** → apna collection → **Runs**
2. **Performance test** / **Performance run** pe jao
3. **Load Profile**: `Fixed` rakho — matlab shuru se aakhir tak fixed users, badhte hue nahi (`Ramp up`, `Spike`, `Peak` options bhi hote hain)
4. **Virtual users**: default 20 hota hai, isko **10** (ya 5) kar do
5. **Duration**: 10 minute nahi, **1–2 minute** kaafi hai
6. **Run** dabao

> **Select File ka option:** bahut baar ML application ki prediction pipeline **batch prediction** karti hai — ek record ki jagah aap CSV/Excel file bhejte ho jisme 50–100 records hain, aur unke against prediction chahiye. Aisi service test karni ho to yahan file select kar sakte ho — matlab 10 users lagatar usi file ke saath service hit kar rahe hain.

**Result:** Dono log windows me load aata hua dikhega — kam-zyada, lekin **dono pods me distribute ho raha hai**. Postman ki blue line **latency** dikhati hai — kahan kam thi, kahan zyada.

> Video me recording ki wajah se system pe load tha, isliye ye clearly nahi dikh paya. **Aapke system pe recording ka pressure nahi hoga, to aap easily dekh paoge.**

> Agar Minikube restart kiya to **service ka port change ho jayega** — Postman me URL update karna mat bhoolna, warna error aayega.

---

## Cluster Band Karna

```bash
# service ko Ctrl+C se interrupt karo, phir:
minikube stop
```

Aur sab kuch clean karna ho to:

```bash
minikube delete --all
```

---

## Image ko Docker Hub / ECR se Pull Karna

Agar aap `imagePullPolicy: Never` wali lines comment out karke **remote repo wali image line** use kar rahe ho, to pehle image ko us repo me push karna zaroori hai:

```bash
docker tag kubernetes-test-app:latest <your-dockerhub-username>/<repo>:<tag>
docker push <your-dockerhub-username>/<repo>:<tag>
```

Push hone ke baad `deployment.yaml` me apne Docker Hub (ya ECR) ka repo path use kar lo.

> Push karne ka detail process is playlist ke **Docker tutorial** me cover ho chuka hai.

---

## Kubernetes ko CI/CD Flow me Kaise Fit Karein

**Purana workflow (jo playlist ke project me tha):**

1. `main` branch pe push karo
2. **GitHub Actions** trigger hota hai
3. Docker image build hoti hai
4. Image **AWS ECR** pe push hoti hai
5. Image **EC2 instance** pe deploy hoti hai aur run hoti hai

**Kubernetes ke saath modified workflow:**

- **Steps 1–4 bilkul same rahenge.** GitHub Actions ab bhi image build karke ECR pe push karega — **yahan koi change nahi.**
- **Change yahan hai:** EC2 instance pe directly deploy karne ki jagah, aap **ek Kubernetes cluster pe deploy karte ho** — jaise AWS pe **EKS**, ya koi bhi managed Kubernetes service.

**Kaise?** Apni CI/CD pipeline (`.github/workflows/aws.yaml`) me ek **naya step add karo** jo `kubectl` CLI (ya GitHub ka Kubernetes action) use karke deployment manifest apply kare, ECR ki latest image ko reference karte hue:

```yaml
- name: Deploy to Kubernetes
  run: kubectl apply -f deployment.yaml
```

Purane project ki YAML file me jahan *"Login to ECR and run Docker image to serve users"* wala EC2 step tha, **wahi step replace hota hai** is Kubernetes deploy step se. Azure ya GCP pe corresponding service ka step aayega.

> Humare case me ek hi file (`deployment.yaml`) daalni hai, kyunki humne **deployment aur service dono usi file me** likh diya hai.

**Isse kya milta hai?** Scalability, fault tolerance, rolling updates — Kubernetes deployment ke saare features.

> Managed services **free tier me nahi aati**, unka kharcha hota hai — isliye project me implement nahi kiya gaya. Demand hone pe aage ke projects me dikhaya jayega.

---

## Real Life me Cluster Kahan Banta Hai

Real world me clusters typically do jagah bante hain:

- **On-premise physical/virtual servers** (expensive, high config — sainkdon GB RAM, kai TB storage, aur ek hired professional jo manage kare)
- **Fully managed Kubernetes service** aapke cloud provider ki taraf se

### On-Premise Servers

**Kaun use karta hai:** Wo organizations jo apne khud ke data centers manage karti hain — **banks, government institutions**. Ye **security ya compliance reasons** se public cloud pe rely nahi kar sakte.

**Kaise:** Jaise humne Minikube use kiya, uske parallel lekin thode advanced tools use hote hain — **kubeadm** aur **Rancher** — physical ya virtual server pe Kubernetes set up karne ke liye.

> Instructor honestly kehte hain: *"Ye kubeadm ya Rancher chalate kaise hain, ye mujhe bhi nahi pata — it is out of my scope as an ML professional. Lekin Kubernetes professionals ko iska idea zaroor hota hai."* Aapko bas ye pata hona chahiye ki inke help se on-prem cluster banta hai.

**Challenge:** Aapko **control plane, nodes — sab khud manage karna padta hai.** Iske liye ek professional hire karna padta hai.

### Managed Kubernetes Services (EKS / AKS / GKE)

**Most companies yahi prefer karti hain.**

| Cloud | Service |
|---|---|
| AWS | **EKS** (Elastic Kubernetes Service) |
| Azure | **AKS** (Azure Kubernetes Service) |
| GCP | **GKE** (Google Kubernetes Engine) |

**Managed ka matlab kya hai?** Part 1 aur 2 me baar-baar aaya ki bahut sara advance-level kaam ek **Kubernetes Administrator / expert professional** se karwaya jata hai. Us professional ki salary **50–60 lakh** tak ho sakti hai. **Agar aap ek startup ho, aap use afford nahi kar paoge.** To aap ye service **cloud pe rent kar lete ho** — Kubernetes aap unke platform pe chalao, aur jo manage karna hai wo unke professionals kar denge.

**Benefits:**

- **Cloud provider control plane handle karta hai** — networking, scaling, sab wo dekh lete hain. Aapko professional hire karne ki zaroorat nahi.
- **Aap sirf apne app ko deploy aur manage karne pe focus karte ho**
- **Doosre cloud services ke saath easy integration** — AWS pe ho to S3, ECR waghera se communication bahut aasan rehta hai

**AWS EKS vs Azure AKS (ek comparison jo notes me hai):**

- **AWS EKS** — advanced, thoda zyada **manual setup** required. Isko chalane ke liye kuch prior understanding chahiye — jaise manual transmission wali car chalana.
- **Azure AKS** — **beginner friendly**, highly automated. Jaise automatic transmission wali easy-to-drive car rent karna.

> **Exception:** Agar aap bank ya koi government authority ho aur data highly confidential rakhna hai, to aap shayad cloud pe na jao — on-prem hi choose karoge.

---

## Summary / Key Takeaways

Poora flow ek nazar me:

1. **App banaya** aur locally test kiya (simple Flask app — ML app bhi ho sakta tha)
2. **Docker Desktop + Docker Hub** ready rakha, dono pe signed in
3. **Dockerfile** banaya (python:3.9-slim base, lightweight rakhne ke liye `.pyc` disable)
4. **Docker image build** ki (`docker build -t <name>:<tag> .`) aur `docker run` karke test bhi ki
5. **deployment.yaml** (Kube Manifest) likhi — jisme `kind: Deployment` aur `kind: Service` dono `---` separator ke saath define kiye. VS Code ka **Kubernetes extension** ye file likhna bahut aasan bana deta hai.
6. **Minikube** download kiya, `minikube start` se **cluster banaya**. Start pe aane wale error ke liye troubleshooting steps: internet check, proxy unset/set, registry nslookup, `minikube stop` + `minikube delete --all` + system restart.
7. **Cluster explore kiya** — `minikube status`, `kubectl get pods -A`, `kubectl get nodes -A`. Realize kiya ki **Kubernetes ke apne components (etcd, API server, scheduler, kube-proxy) khud microservices/pods hain jo Docker ke upar chalte hain.**
8. **Extra worker nodes** chahiye to `minikube start --nodes=2`
9. **Image cluster me load ki** — `minikube image load <name>:<tag>`. Aur ye bhi dekha ki repo (Docker Hub / ECR) se fetch karna ho to manifest me kya badalta hai.
10. **Deploy kiya** — `kubectl apply -f deployment.yaml`, delete kiya — `kubectl delete -f deployment.yaml`
11. **Self-healing dekha** — pod manually delete kiya, Kubernetes ne **~15 second me naya pod spin off** kar diya. `replicas: 2` maanga hai to 2 rahenge hi.
12. **Service se single endpoint** mila — `minikube service <app-name>` — chahe kitne bhi pods me service deployed ho.
13. **Live debugging seekhi** — `ImagePullBackOff` ka reason tha `imagePullPolicy: Never` comment out reh jaana. **Rule:** ya to local image + `imagePullPolicy: Never`, ya remote repo path — beech ka kuch nahi.
14. **Minikube dashboard** dekha (monitoring feature, production me use nahi hota — wahan Prometheus + Grafana)
15. **Endpoints/services** check karna aur **Ingress (andar aane ka gateway) vs Egress (bahar jaane ka gateway)** ka basic idea
16. **Load balancing test kiya Postman se** — fixed load profile, 10 virtual users, 1 minute, aur dono pods ke live logs (`kubectl logs -f`) me load distribute hota dekha
17. **CI/CD me Kubernetes fit kiya** — GitHub Actions → ECR tak sab same, bas EC2 deploy step ki jagah `kubectl apply` wala Kubernetes deploy step
18. **Production reality samjhi** — on-prem (kubeadm, Rancher — banks/government) vs managed services (EKS/AKS/GKE — most companies). **Minikube kabhi production me use nahi hota.**

> **Bottom line:** In do videos ka knowledge ek MLOps professional ke liye Kubernetes pe kaam karne ke liye **more than enough** hai. Chahe aap kisi badi organization me jao jo already Kubernetes implement kar rahi hai (wahan aap easily pick up kar loge), ya kisi startup me jo abhi explore karna shuru kar raha hai — aap scratch se kaam karne ke liye ready ho.
