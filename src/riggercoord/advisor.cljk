(ns riggercoord.advisor
  "Rigging Coordination Advisor — the advisor named in this
  repository's README, proposing a job-site scheduling/logistics
  coordination operation (log a work record, schedule a crew
  operation, flag a safety concern, coordinate a supply order) from a
  site roster, crew roster and job-site schedule. Swappable mock/llm;
  the advisor ONLY proposes — `riggercoord.governor` checks
  site/worker registration, the closed op-allowlist and
  scope-exclusion independently, and always escalates safety-concern
  flags, above-threshold supply orders and low-confidence proposals.
  This actor coordinates JOB-SITE SCHEDULING/LOGISTICS ONLY — it
  never performs rigging or cable-splicing work and never proposes to
  finalize a load-rigging/lift-readiness certification decision or
  override a site-safety officer's judgment. Modeled on
  cloud-itonami-isco-7121's advisor.

  A proposal: {:op :log-work-record|:schedule-crew-operation|
               :flag-safety-concern|:coordinate-supply-order
               :effect :propose :site-id str :worker-id str? :cost
               number? :stake kw :confidence n :rationale str}")

(defprotocol Advisor
  (-advise [advisor store request] "request -> proposal map"))

(defn- infer [_store {:keys [op stake site-id worker-id cost task materials
                             concern-type severity description time-window
                             progress-notes]
                      :as request}]
  (cond-> {:op op
           :effect :propose
           :site-id site-id
           :stake (or stake :low)
           :confidence (case (or stake :low) :high 0.7 :medium 0.85 :low 0.95)
           :rationale (str "proposed " (name op) " for site " site-id)}
    worker-id (assoc :worker-id worker-id)
    (some? cost) (assoc :cost cost)
    task (assoc :task task)
    materials (assoc :materials materials)
    concern-type (assoc :concern-type concern-type)
    severity (assoc :severity severity)
    description (assoc :description description)
    time-window (assoc :time-window time-window)
    progress-notes (assoc :progress-notes progress-notes)))

(defn mock-advisor []
  (reify Advisor
    (-advise [_ store request] (infer store request))))

(def ^:private system-prompt
  "You are a rigging and cable-splicing crew scheduling/logistics
   coordination advisor. Given a request, propose an :op
   (:log-work-record, :schedule-crew-operation, :flag-safety-concern
   or :coordinate-supply-order ONLY — no other op exists), the
   :site-id, an honest :confidence and a :stake. You coordinate
   job-site scheduling and logistics ONLY: never propose to finalize
   a load-rigging/lift-readiness certification decision, never
   propose to override a site-safety officer's judgment, and never
   propose an op outside the closed allowlist above. The governor
   checks site/worker registration and scope independently.
   Safety-concern flags and above-threshold supply orders always
   require human sign-off regardless of confidence.")

(defn- parse-proposal [content]
  (try
    (let [p (read-string content)]
      (if (map? p)
        (assoc p :effect :propose)
        {:op :unknown :effect :propose :confidence 0.0 :stake :high
         :rationale "unparseable LLM response"}))
    (catch #?(:clj Exception :cljs js/Error) _
      {:op :unknown :effect :propose :confidence 0.0 :stake :high
       :rationale "LLM response parse failure"})))

(defn llm-advisor
  [chat-model model-generate-fn gen-opts]
  (reify Advisor
    (-advise [_ _store request]
      (let [msgs [{:role :system :content system-prompt}
                  {:role :user :content (str "operation request: " (pr-str request))}]
            resp (model-generate-fn chat-model msgs gen-opts)]
        (parse-proposal (:content resp))))))
