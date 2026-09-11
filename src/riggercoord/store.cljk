(ns riggercoord.store
  "SSoT for the ISCO-08 7215 rigging and cable-splicing crew
  scheduling/logistics coordination actor (itonami actor pattern,
  ADR-2607011000 / CLAUDE.md Actors section; README's 'Robotics
  premise' — a job-site scheduling/logistics coordination robot
  proposes crew scheduling, work-record logging, safety-concern flags
  and rigging-equipment supply-order coordination under this
  advisor/governor pair, which never dispatches hardware itself, never
  performs rigging or cable-splicing work, and never finalizes a
  load-rigging/lift-readiness certification decision or overrides a
  site-safety officer's judgment). Modeled on
  cloud-itonami-isco-7121's roofcoord.store.

  Domain:

    site    — a registered rigging job site (:site-id, :name, :address).
    worker  — a registered crew member {:worker-id :site-id :name
              :role}, belonging to exactly one registered site (e.g. a
              rigger or cable splicer).
    record  — a committed operating record (a logged work record,
              scheduling proposal, safety-concern flag or supply-order
              coordination entry) — written ONLY via commit-record!.
              This actor coordinates job-site scheduling/logistics
              ONLY — a `record` is a coordination artifact, never a
              load-rigging/lift-readiness certification act or a
              site-safety-officer judgment override.
    ledger  — append-only audit trail, commit or hold."
  )

(defprotocol Store
  (site [s site-id])
  (worker [s worker-id])
  (records-of [s site-id])
  (ledger [s])
  (register-site! [s site])
  (register-worker! [s w])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (site [_ site-id] (get-in @a [:sites site-id]))
  (worker [_ worker-id] (get-in @a [:workers worker-id]))
  (records-of [_ site-id] (filter #(= site-id (:site-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-site! [s site]
    (swap! a assoc-in [:sites (:site-id site)] site) s)
  (register-worker! [s w]
    (swap! a assoc-in [:workers (:worker-id w)] w) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:sites {} :workers {} :records [] :ledger []}
                                   seed)))))
