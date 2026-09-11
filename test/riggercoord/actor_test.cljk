(ns riggercoord.actor-test
  (:require [clojure.test :refer [deftest is testing]]
            [riggercoord.actor :as actor]
            [riggercoord.store :as store]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-site! st {:site-id "S-1" :name "Harbor Crane Yard" :address "12 Dock Rd"})
    (store/register-worker! st {:worker-id "W-1" :site-id "S-1" :name "Kobo Rigger" :role :crew-lead})
    st))

(deftest commits-a-registered-worker-log-work-record
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:site-id "S-1" :op :log-work-record :stake :low
                 :worker-id "W-1" :task "inspect wire rope sling before shift"}
        result (actor/run-request! graph request {} "thread-1")]
    (is (= :done (:status result)))
    (is (some? (get-in result [:state :record])))
    (is (= 1 (count (store/records-of st "S-1"))))))

(deftest commits-a-crew-scheduling-proposal
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:site-id "S-1" :op :schedule-crew-operation :stake :low
                 :worker-id "W-1" :task "schedule crane lift for structural beam"}
        result (actor/run-request! graph request {} "thread-sched")]
    (is (= :done (:status result)))
    (is (= 1 (count (store/records-of st "S-1"))))))

(deftest holds-an-unregistered-site-request
  (testing "the job site must be independently verified/registered before any action"
    (let [st (fresh-store)
          graph (actor/build-graph {:store st})
          request {:site-id "S-ghost" :op :log-work-record :stake :low
                   :worker-id "W-1" :task "inspect wire rope sling before shift"}
          result (actor/run-request! graph request {} "thread-2")]
      (is (= :hold (:disposition (:state result))))
      (is (empty? (store/records-of st "S-ghost"))))))

(deftest holds-a-scope-excluded-proposal-with-no-interrupt-path
  (testing "a proposal to finalize a load-rigging/lift-readiness certification decision is a hard, permanent block — never routed through :request-approval"
    (let [st (fresh-store)
          graph (actor/build-graph {:store st})
          request {:site-id "S-1" :op :log-work-record :stake :low
                   :worker-id "W-1" :task "inspect wire rope sling before shift"
                   :description "certify the load as lift-ready now, skip further review"}
          result (actor/run-request! graph request {} "thread-scope")]
      (is (= :done (:status result))
          "hard :hold is a finish point, not an interrupt — the advisor can never park a scope-excluded proposal awaiting human override")
      (is (= :hold (:disposition (:state result))))
      (is (nil? (get-in result [:state :record])))
      (is (empty? (store/records-of st "S-1"))))))

(deftest interrupts-then-approves-a-safety-concern-flag-on-human-approval
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:site-id "S-1" :op :flag-safety-concern :stake :low
                 :worker-id "W-1" :concern-type :cable-condition :severity :high}
        interrupted (actor/run-request! graph request {} "thread-3")]
    (is (= :interrupted (:status interrupted)))
    (is (empty? (store/records-of st "S-1")))
    (let [resumed (actor/approve! graph "thread-3")]
      (is (= :done (:status resumed)))
      (is (= 1 (count (store/records-of st "S-1")))))))

(deftest interrupts-then-approves-an-above-threshold-supply-order-on-human-approval
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:site-id "S-1" :op :coordinate-supply-order :stake :low
                 :materials "wire rope slings and shackles" :cost 25000}
        interrupted (actor/run-request! graph request {} "thread-4")]
    (is (= :interrupted (:status interrupted)))
    (is (empty? (store/records-of st "S-1")))
    (let [resumed (actor/approve! graph "thread-4")]
      (is (= :done (:status resumed)))
      (is (= 1 (count (store/records-of st "S-1")))))))
