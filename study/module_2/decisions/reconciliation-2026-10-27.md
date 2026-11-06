# Module 2 decision reconciliation — updated 5 November 2026

The six topic pages and the decision register below state the Module 2 decisions. Q01–Q07 and PC08 remain settled. S25–S37 incorporate the complete package and catalog directions through 5 November 2026: canonical service names in `services` are immutable post-launch once visits are recorded (D01); admin-only MVP delivery scope is reaffirmed (receptionist role and authorization follow in post-MVP phase); ordinary completed care is never blocked by patient debt; future receptionists are blocked from selling packages if any patient package-linked invoice is unsettled; package naming direction uses `package_catalog` for definition templates and `packages` for purchased entitlements; package upgrades are locked to a narrow same-complaint, same-category 10→26 visit extension (+16 visits, current long package price minus original invoiced charge floored at zero, parent package start-date review anchor preserved, terminal parent Completed or Closed on/after review date required, invoice-level refund context displayed nonblockingly, daily visit limit copied at purchase, one upgrade per parent with terminal finality, and calculated-zero upgrade invoices followed by a separate Mark as paid step). The 26-visit ceiling, 1-per-day long-offering limit, Sunday review arithmetic, fixed 30-consecutive-day review duration, review clock anchored to package start date ($T$ or $T+1$), configurable daily visit cap (default 1, admin mutable, MVP change-audit deferred), strict closure permanence and irreversibility even for clerical mistakes, renewal debt assist-not-block, justification notes strictly immutable for everyone, admin oversight without alerts/pings, two persistent dashboard query lists, payment-independent package activation, mandatory post-review justification notes in MVP, active package blocking complaint closure, counter recording sequence without historical treatment timestamp, two-stage review UX, role-based closure (admin anytime, receptionist strictly on overdue status), the 7 closure reason presets, backend date validation on "Patient Stopped Attending", post-closure factual summary, explicit rejection of queuing, strict same-complaint purchase blocking, N+1 renewal start, and independent routine/rehab pricing remain settled. Questionnaire Q01–Q40 is complete. Parked technical audits (concurrency, race conditions, server-controlled dates/prices, retry idempotency, write paths) remain open as engineering tasks. Original historical wording is evidence only. This record does not establish deployed schema.

## Source inventory and precedence

| Source | Archived record / evidence | Contribution |
| --- | --- | --- |
| S01 | [clinic_checkout_billing_decision_log_2026-10-18.md](history/clinic_checkout_billing_decision_log_2026-10-18.md) | Earlier checkout/payment baseline, including later §15 |
| S02 | [clinic_billing_decision_log_2026-10-23.md](history/clinic_billing_decision_log_2026-10-23.md) | Parent register and original O01–O21 |
| S03 | [clinic_module2_successor_addendum_2026-10-23_1726_IST.md](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md) | Later flat-visit, source, and package decisions |
| S04 | Historical repository schema cited by the existing workbook, not inspected as live deployment | Historical repository schema cited by the existing workbook, not inspected as live deployment |
| S05 | Original workbook's in-chat attribution; retained historical context, not a new transcript | Original workbook's in-chat attribution; retained historical context, not a new transcript |
| S06 | [clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md](history/clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md) | Initial October 25 ledger and N01–N27 |
| S07 | [clinic_module2_successor_addendum_2026-10-25_171327_IST.md](history/clinic_module2_successor_addendum_2026-10-25_171327_IST.md) | Intervening append/voiding permissions, later withdrawn for MVP |
| S08 | [clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md](history/clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md) | G01–G15 and source/financial integrity risks |
| S09 | [clinic_module2_scope_decision_addendum_2026-10-25.md](history/clinic_module2_scope_decision_addendum_2026-10-25.md) | Scope reversal and later payment/package continuation |
| S10 | [clinic_module2_waiver_catalog_addendum_2026-10-25.md](history/clinic_module2_waiver_catalog_addendum_2026-10-25.md) | Waived terminal status; single-clinic services and future migration |
| S11 | [clinic_module2_running_log_and_followup_2026-10-25.md](history/clinic_module2_running_log_and_followup_2026-10-25.md) | Exam-only attendance, mandatory clinical re-examination, calendar bands |
| S12 | [clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md](history/clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md) | Repayments, source cardinality boundaries, clinical-first exception, idempotency requirement |
| S13 | [clinic_module2_post_addendum_decisions_2026-10-25.md](history/clinic_module2_post_addendum_decisions_2026-10-25.md) | Methods, split amounts, waiver reason/flag, no backdating, separate settlement |
| S14 | [clinic_module2_running_decision_log_2026-10-26_1130_IST.md](history/clinic_module2_running_decision_log_2026-10-26_1130_IST.md) | D26-01–D26-15 and the appended payment field/deletion decisions |
| S15 | [module2_mvp_chat_decision_log_2026-10-26.md](history/module2_mvp_chat_decision_log_2026-10-26.md) | Earlier reconstruction plus later D-26–D-44 reconciliation |
| S16 | [chatgpt-full-transcript-1793041621482.json](history/chatgpt-full-transcript-1793041621482.json) | 176-message primary export; displaced opening replies and incomplete Show more text |
| S17 | [User reconciliation answers and corrections](history/clinic_module2_reconciliation_resolutions_2026-10-27.md) | Q01–Q04 resolutions/corrections; Q05–Q07 question explanations |
| S18 | [Current decision authority](history/clinic_module2_current_decision_authority_2026-10-27.md) | Current files must agree; earlier label choice subsequently superseded by S21 |
| S19 | [Clinical preview and fee visibility](history/clinic_module2_clinical_preview_fee_visibility_2026-10-27.md) | Q05 settled; Q06 discussion |
| S20 | [Admin catalog authority](history/clinic_module2_admin_catalog_authority_2026-10-27.md) | Admin-only prices and availability |
| S21 | [Status names and standalone identity](history/clinic_module2_status_names_and_standalone_identity_2026-10-27.md) | Approved UNPAID/PARTIALLY_PAID/PAID/WAIVED; Q06 service ID with joined name; all seven review questions settled |
| S22 | [Consolidated seven-answer history log](history/clinic_module2_seven_review_answers_2026-10-27.md) | Compact final answers and owner explanations for Q01–Q07; no new policy |
| S23 | [Examination/lapse fee meaning](history/clinic_module2_examination_fee_semantics_2026-10-27.md) | Billing-only fees; independent first/recurrent fee-only checkout; attendance recency and five-day no-fee clarification |
| S24 | [Collection authority and delivery phases](history/clinic_module2_reception_payment_authority_2026-10-29.md) | PC08 resolved; future reception initial/later receipts, admin-only waiver, admin-only current phase and later reception/RLS |
| S25 | [Package handover, 28 October](history/clinic_packages_handover_2026-10-28.md) | User decisions, reported owner requirements, proposals and open questions through 28 October |
| S26 | [Package addendum, 28–29 October](history/clinic_packages_addendum_2026-10-28_to_2026-10-29.md) | Subsequent directions and changes, owner-confirmation gaps, proposals and unresolved conflicts through 29 October |
| S27 | [clinic_packages_discussion_patch_2026-11-02_181036_IST.md](history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md) | Third cumulative package record: 26 hard cap, 1/day, Sunday review arithmetic, closure authority, registration clock, payment gating reopened |
| S28 | [package_decision_log_2026-11-02_voice_session.md](history/package_decision_log_2026-11-02_voice_session.md) | Fourth package record: 7 closure reasons, "Patient Stopped Attending" date rule, post-closure summary, queuing rejected, replacement requires prior termination; S30 later settled same-day start |
| S29 | [clinic_package_decision_log_2026-11-03_pre_questionnaire.md](history/clinic_package_decision_log_2026-11-03_pre_questionnaire.md) | Fifth package record: payment-independent activation, fixed 30-day review confirmed, mandatory post-review notes in MVP, active package blocks complaint closure, counter recording sequence |
| S30 | [package_module_questionnaire_reconstructed_2026-11-03.md](history/package_module_questionnaire_reconstructed_2026-11-03.md) | Sixth package record: Q01–Q16 settled (package start date anchor, daily visit cap default 1, closure irreversibility, debt renewal assist-not-block, justification immutability, 2 query lists), Q17 dropped, Q18–Q40 unanswered |
| S31 | [package_module_Q18-Q40_decision_addendum_2026-11-03.md](history/package_module_Q18-Q40_decision_addendum_2026-11-03.md) | Seventh package record: Q18–Q31 settled (single combined invoice, mistaken package refund/waiver workflows, Cash/UPI only, entitlement survival on waiver, admin-only pricing, no price override/free sale, no disable UI, unconfigured price block, internal display); DR-01/DR-02 logged; Q32–Q40 active baseline |
| S32 | [package_module_final_voice_decision_addendum_2026-11-03_Q32_onwards.md](history/package_module_final_voice_decision_addendum_2026-11-03_Q32_onwards.md) | Eighth package record: Q32–Q40 settled (paper packages on paper, digitization deferred to separate module, Q37 flat visit coverage, Q38 nonblocking warning, Q39 general descending invoice dues, Q40 committed care unchanged); Q01–Q40 questionnaire formally complete |
| S33 | [package_call_transcript_discussion_log_2026-11-03_2231_IST.md](history/package_call_transcript_discussion_log_2026-11-03_2231_IST.md) | Ninth package record: confirmed admin-only refunds (T73), freeze/pause rejected for MVP, Routine=Regular synonymy, full entitlement on partial payment, and 10→26 package upgrade proposal (S33) |
| S34 | [package_upgrade_design_discussion_log_2026-11-04_0220_IST.md](history/package_upgrade_design_discussion_log_2026-11-04_0220_IST.md) | Tenth package record: 10→26 upgrade economics, linked extension concept, Day-30 window and zero price floor |
| S35 | [response.md](history/claude/response.md) | Eleventh package record: Claude scratch-model upgrade audit and technical findings (historical evidence only) |
| S36 | [package_upgrade_prototype_and_test_results_2026-11-04.md](history/claude/package_upgrade_prototype_and_test_results_2026-11-04.md) | Twelfth package record: Claude scratch prototype and concurrency/guard test report (scratch-model evidence) |
| S37 | [catalog_identity_discussion_2026-11-04.md](history/catalog_identity_discussion_2026-11-04.md) | Thirteenth package record: canonical service name immutability (D01), locked package upgrade rules, admin-only MVP, ordinary care unblocked by debt, future reception debt rules, package_catalog naming |

S01–S09 preserve the existing workbook source IDs. S10–S37 extend them. S17–S21 and S23–S24 record the subsequent user clarifications in this chat and are not part of S16. Earlier October 21/22 and O06 archived records remain linked by the owning topics and unchanged in history. The existing October 25 chronology guide remains a historical reconciliation snapshot.

Use subject-specific explicit approvals, not a blanket newest-file-wins rule. S14 is internally incremental: its appended §8 follows its initial checkpoint. S15 preserves earlier reconstruction alongside later decisions; S16 directly supports only the supplied export. A later assistant proposal is not approval. Original decision IDs are source-qualified because similarly named IDs in different logs are not the same record.

The export capture is 2026-10-26T19:07:01.478Z (27 October at 00:37:01.478 IST), not a timestamp for every utterance. M001–M008 are displaced before the handoff at M009; some text ends with Show more. Do not pair short replies to proposals by invented adjacency or reconstruct missing text. M013/M031 reports about actual schema remain unverified reports.

## Archive manifest

| Source | Original location | Archive location | Disposition | Bytes | SHA-256 |
| --- | --- | --- | --- | --- | --- |
| S10 | upcoming/clinic_module2_waiver_catalog_addendum_2026-10-25.md | [history/clinic_module2_waiver_catalog_addendum_2026-10-25.md](history/clinic_module2_waiver_catalog_addendum_2026-10-25.md) | Move after verification | 8669 | 32DA74D3B5133EA336D3253B864049972E9C91E69E340F9A7CF7B82A528F462D |
| S11 | upcoming/clinic_module2_running_log_and_followup_2026-10-25.md | [history/clinic_module2_running_log_and_followup_2026-10-25.md](history/clinic_module2_running_log_and_followup_2026-10-25.md) | Move after verification | 10658 | 423193AAAEB9C51F630CD36B3083B6F07556F3AAA2DB66F363A7A02E81141A81 |
| S12 | upcoming/clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md | [history/clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md](history/clinic_module2_six_questions_three_ideas_settled_addendum_2026-10-25.md) | Move after verification | 17570 | B80A06DC6367C804F2A0F19184E840224B0CD6182A17178A49E2E34ADEA54F8A |
| S13 | upcoming/clinic_module2_post_addendum_decisions_2026-10-25.md | [history/clinic_module2_post_addendum_decisions_2026-10-25.md](history/clinic_module2_post_addendum_decisions_2026-10-25.md) | Move after verification | 14978 | 51D2A365D921C896E7973E6BF5E6E2DF307087719DFEE1915EFAE131C0FDBB3E |
| S14 | upcoming/clinic_module2_running_decision_log_2026-10-26_1130_IST.md | [history/clinic_module2_running_decision_log_2026-10-26_1130_IST.md](history/clinic_module2_running_decision_log_2026-10-26_1130_IST.md) | Move after verification | 31679 | 5D6E97C661DB61721DA8036E2FC112FBCCC6DF1D065359FAC870FCADC6C09837 |
| S15 | dump/module2_mvp_chat_decision_log_2026-10-26.md | [history/module2_mvp_chat_decision_log_2026-10-26.md](history/module2_mvp_chat_decision_log_2026-10-26.md) | Copy; retain dump original | 46269 | 06C7BCDC0C97355A560B1ED568F92902DF1B5285532C3687581DF0F3B8D1B4A2 |
| S16 | dump/chatgpt-full-transcript-1793041621482.json | [history/chatgpt-full-transcript-1793041621482.json](history/chatgpt-full-transcript-1793041621482.json) | Copy; retain dump original | 71930 | 29B8C59A649B54A1C4E6F99F66B7FE1F231BF69B949BA75920C9FC286515C332 |
| S27 | Downloads/clinic_packages_discussion_patch_2026-11-02_181036_IST.md | [history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md](history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md) | Copy; retain original | 61484 | 5CC6594D2670145F6CAE4B76A1C8FF7A769CF310A79974CE2BD4E389CAF825C6 |
| S28 | Downloads/package_decision_log_2026-11-02_voice_session.md | [history/package_decision_log_2026-11-02_voice_session.md](history/package_decision_log_2026-11-02_voice_session.md) | Copy; retain original | 15033 | 447F5CDB22CEFE78F72CD709B8CFC8BB64D3CC32DE0B08A8198FA00D6C57BE0F |
| S29 | Downloads/clinic_package_decision_log_2026-11-03_pre_questionnaire.md | [history/clinic_package_decision_log_2026-11-03_pre_questionnaire.md](history/clinic_package_decision_log_2026-11-03_pre_questionnaire.md) | Copy; retain original | 21594 | 4C046D4F06F9F98C30AA3E95A9AA230F7717D80CF4B6CDA07BDBB480E22FB79C |
| S30 | Downloads/package_module_questionnaire_reconstructed_2026-11-03.md | [history/package_module_questionnaire_reconstructed_2026-11-03.md](history/package_module_questionnaire_reconstructed_2026-11-03.md) | Copy; retain original | 51547 | F5557B48936F104DB03320DD07EF309701D237F46BB8E281661C996593C66C32 |
| S31 | Downloads/package_module_Q18-Q40_decision_addendum_2026-11-03.md | [history/package_module_Q18-Q40_decision_addendum_2026-11-03.md](history/package_module_Q18-Q40_decision_addendum_2026-11-03.md) | Copy; retain original | 51718 | 7F39938506E8F7EA76534183382D26AB3B3BF1EF946D50C79470A7863C5438F2 |
| S32 | Downloads/package_module_final_voice_decision_addendum_2026-11-03_Q32_onwards.md | [history/package_module_final_voice_decision_addendum_2026-11-03_Q32_onwards.md](history/package_module_final_voice_decision_addendum_2026-11-03_Q32_onwards.md) | Copy; retain original | 70722 | 5676E1A64386A2FDE510D09CBC25845C37560872B0A7C0AD7765C9F883C00C0C |
| S33 | Downloads/package_call_transcript_discussion_log_2026-11-03_2231_IST.md | [history/package_call_transcript_discussion_log_2026-11-03_2231_IST.md](history/package_call_transcript_discussion_log_2026-11-03_2231_IST.md) | Copy; retain original | 11573 | E994CEF6399772334F9FF949388C2764D513C255C4587831CE5A4B256206AA95 |
| S34 | upcoming/package_upgrade_design_discussion_log_2026-11-04_0220_IST.md | [history/package_upgrade_design_discussion_log_2026-11-04_0220_IST.md](history/package_upgrade_design_discussion_log_2026-11-04_0220_IST.md) | Move after verification | 19725 | D034827E4A7B6CEAF744D9FDFFC9DCEB8BEDB6FDFE211F34F2A949137CD361A6 |
| S35 | upcoming/claude/response.md | [history/claude/response.md](history/claude/response.md) | Move after verification | 18466 | 23411AEF565DEBF3648B09A534D36E549809DC0954F6CC8E27ED8DED04FE5AC1 |
| S36 | upcoming/claude/package_upgrade_prototype_and_test_results_2026-11-04.md | [history/claude/package_upgrade_prototype_and_test_results_2026-11-04.md](history/claude/package_upgrade_prototype_and_test_results_2026-11-04.md) | Move after verification | 59462 | 9D5CBE36B661A3E7FA1FE52E5C807ED836D1AFB93B5AB753AC9C6F1BBC7E7398 |
| S37 | upcoming/catalog_identity_discussion_2026-11-04.md | [history/catalog_identity_discussion_2026-11-04.md](history/catalog_identity_discussion_2026-11-04.md) | Move after verification | 77508 | 95401D199F9B7EC471E2932C5F0242AE504D3273BDBD324872D68AF5E68507C6 |

All basenames were verified at preflight. Sources S10–S14 and S34–S37 are moved byte-for-byte; dump log/JSON and Downloads logs are copied and their originals retained. The intermediate timing log `clinic_packages_replacement_and_timing_session_2026-11-03.md` (6,992 bytes, SHA-256 `663674697F8C43093121545464FC3DF47B95C370A3D2CD9585671F11C8EB8E8A`) was evaluated during preflight; its five specific decisions (same-complaint blocking, same-day vs next-day start dates, N+1 renewal start, 7 closure reasons, post-closure summary) are identical to and fully covered by S28, S29, and S30, and it is safely deleted during Step 10 without archiving. Original metadata paths inside preserved sources are historical. Use this manifest for current locations. Later changes to a dump original do not mutate its archived snapshot.

The 11 pre-existing history files and the research document remain byte-for-byte unchanged. Research is background material, not user-approved clinic policy. No log is deleted merely because it overlaps another.

### Existing historical archive

| Preserved original | Bytes | SHA-256 |
| --- | --- | --- |
| [2026-10-25-chronology-and-overlap.md](history/2026-10-25-chronology-and-overlap.md) | 14149 | 9D218728328ED788307C1F3772B742032ED35F2C79250C0BDA6B1DD7CCDD9CD7 |
| [clinic_billing_decision_addendum_2026-10-21.md](history/clinic_billing_decision_addendum_2026-10-21.md) | 3615 | A1854C2047193556244218A477697F63B56AD9AEFEF171C9DA8B4234B7117F60 |
| [clinic_billing_decision_log_2026-10-23.md](history/clinic_billing_decision_log_2026-10-23.md) | 64200 | 118F8E6B01243B838F6573A2B86EF9959B7F3CB6F069277B14621C1F9A9946D8 |
| [clinic_checkout_billing_decision_log_2026-10-18.md](history/clinic_checkout_billing_decision_log_2026-10-18.md) | 55745 | 222784105F56A98D9153509EB7CCFA658DD131D3CCE1058BDCCA11E025E87385 |
| [clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md](history/clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md) | 32841 | 64F8B46C3BB392BEF3D7CEA849CA97AA0284E64AC846D195D50720D0AF372FAC |
| [clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md](history/clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md) | 43499 | EEE8309D367A5958D9CDC7B7C6A9F9395A6FD4942A74190AB1C7F75E393977A7 |
| [clinic_module2_scope_decision_addendum_2026-10-25.md](history/clinic_module2_scope_decision_addendum_2026-10-25.md) | 21077 | C4526CA3CC76923F8680FDB0056923B221B0CA00AD270E2FDB806887DE3C12F5 |
| [clinic_module2_successor_addendum_2026-10-23_1726_IST.md](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md) | 34785 | 5CF986D88C5EBEE3FD3FD3B4DE8EF53F7851EFAF3705DFF7AF280F6AC9E13817 |
| [clinic_module2_successor_addendum_2026-10-25_171327_IST.md](history/clinic_module2_successor_addendum_2026-10-25_171327_IST.md) | 15345 | 98C1B60CFA861D69428847739D1F53ABB3C6505D607628DD6356D9E3E4286D34 |
| [clinic-billing-decision-log-2026-10-22.md](history/clinic-billing-decision-log-2026-10-22.md) | 25547 | 9A5A94579DAEF5F0363736B0CDBF9C8F9D2C0E9C9E3786BD4687D8BA95902ADD |
| [O06_service_catalog_branch_configuration_decision_brief_2026-10-23.md](history/O06_service_catalog_branch_configuration_decision_brief_2026-10-23.md) | 31162 | BDA4F055F44BCAE53E2DC8C21A42BDED0423626CC8BDE66F19951301030335F0 |

## Subsequent user clarification

S17 records Q01–Q04 resolutions/corrections. S19 subsequently resolves Q05; S20 settles Q07; S21 subsequently settles Q06. The status-rename attribution and clinical-first exception were errors in the earlier reading, not proven deliberate policy reversals by the user. Existing archived sources and dump originals are unchanged.

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S17 | [clinic_module2_reconciliation_resolutions_2026-10-27.md](history/clinic_module2_reconciliation_resolutions_2026-10-27.md) | 6535 | 2604D4DF6CBD1CDCF9174B208CF569FEA7987F8BF748B771E0517B64C4347487 |

S18 establishes one consistent current rule per settled subject; its earlier status-label choice is subsequently superseded by S21. S19 subsequently settles Q05; S20 settles Q07; S21 subsequently settles Q06.

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S18 | [clinic_module2_current_decision_authority_2026-10-27.md](history/clinic_module2_current_decision_authority_2026-10-27.md) | 2549 | 452CABC381939E1663C177F8D0A6864352A2B002CFE50BD2AB189598B2168826 |

S19 settles Q05: the pre-issuance preview is clinical; automatic fees first appear on the returned issued invoice. S20 settles Q07; S21 subsequently settles Q06.

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S19 | [clinic_module2_clinical_preview_fee_visibility_2026-10-27.md](history/clinic_module2_clinical_preview_fee_visibility_2026-10-27.md) | 1976 | 811730FE0B8C0755A44006EEF3C404710CB94E40D07A3CB76B59DF1C9C7C83EB |

S20 settles Q07: only admin may edit master catalog prices and availability. Q06 is subsequently settled by S21.

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S20 | [clinic_module2_admin_catalog_authority_2026-10-27.md](history/clinic_module2_admin_catalog_authority_2026-10-27.md) | 1270 | 232D0DC1E965FB215007F54A57518A1E4F201D456DC05D6CBB269B47809EFBBD |

S21 explicitly approves UNPAID/PARTIALLY_PAID/PAID/WAIVED and settles Q06 as service_id with a joined catalog name,no copied clinical name. All seven review questions are settled; technical audits remain unfinished.

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S21 | [clinic_module2_status_names_and_standalone_identity_2026-10-27.md](history/clinic_module2_status_names_and_standalone_identity_2026-10-27.md) | 2456 | 010FA190E80A264CC7AAAB35BF35363C577D0E9286445F118CB3CD284186ACE6 |

S22 is the compact consolidated history record of all seven final answers and their explanations. It adds no new business policy. The current topic pages and tracker were checked against it.

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S22 | [clinic_module2_seven_review_answers_2026-10-27.md](history/clinic_module2_seven_review_answers_2026-10-27.md) | 3359 | A498F49A71A28973092CB5F3AB6E09A57F95D9BEEE737F827BDD55F497B10169 |

S23 records the owner's later fee clarification: examination/lapse fees are independent billing events, not clinical examination records. First and recurrent fee-only checkout follow the same rules; fee-only attendance resets recency. A five-day return creates no examination/lapse fee event. Current rules below incorporate this clarification; earlier historical sources remain unchanged.

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S23 | [clinic_module2_examination_fee_semantics_2026-10-27.md](history/clinic_module2_examination_fee_semantics_2026-10-27.md) | 2690 | 2305D6FEC7D8065702AA08C6C033B8A22EDC7791B5C0F0D519160954B7118874 |

S24 records the 29 October owner decision. Reception may resume initial collection after interruption and may record later receipts, including after Pay Later or on later days. Only admin may waive. S24 originally placed reception in a later MVP phase. S37 §9 and §22 supersede that timing: MVP is admin-only and reception/RLS follow post-MVP. S24 still resolves PC08 collection permissions for that future role.

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S24 | [clinic_module2_reception_payment_authority_2026-10-29.md](history/clinic_module2_reception_payment_authority_2026-10-29.md) | 2265 | 1A0F5B458394277F6E0C6DF144CEAF3758785FD391EBDAA4963A356DE282C6F6 |

S25–S37 are the thirteen package and catalog records. S27 incorporates the post-addendum discussion (26 hard cap, 1/day limit, Sunday review arithmetic, closure authority, registration clock). S28 incorporates the 2 November voice session decisions (D01–D14: 7 closure reasons, "Patient Stopped Attending" date rule, post-closure summary, queuing rejected, replacement requires prior termination; S30 later settled same-day start). S29 incorporates the 3 November pre-questionnaire session decisions (D01–D08: payment-independent activation, fixed 30-day review duration settled, per-visit notes for future reception, optional for admin, active packages blocking complaint closure, and counter recording sequence). S30 incorporates the 3 November reconstructed final questionnaire decisions (Q01–Q16: review clock starts on package start date, configurable daily cap default 1, closure irreversibility for clerical errors, renewal debt assist-not-block, justification notes immutable, admin oversight without alerts, 2 persistent query lists, nonpayment independence; Q17 dropped). S31 incorporates the 3 November Q18–Q40 addendum (Q18–Q31: single combined invoice for package checkouts, mistaken package refund/waiver workflows, Cash/UPI only, entitlement survival on full waiver, admin-only pricing, no price overrides or zero-price sales, unconfigured price block, internal display; DR-01/DR-02 logged). S32 incorporates the 3 November final voice addendum (Q32–Q40: paper packages on paper, digitization deferred to separate module, Q37 flat visit coverage, Q38 nonblocking warning, Q39 general descending invoice dues, Q40 committed care unchanged), formally concluding the 40-question questionnaire. S33 incorporates the 3 November call transcript and follow-up discussion log (settles T73 with admin-only refund authorization, rejects package freeze/pause for MVP, confirms Routine=Regular synonymy and full entitlement on partial payment, and documents the 10→26 visit package upgrade proposal under design). S34 incorporates the 4 November package upgrade design discussion (10→26 upgrade economics, linked extension concept, Day-30 window, zero price floor). S35 incorporates the Claude technical response and architecture review as technical reference and scratch-model evidence only. S36 incorporates the Claude scratch prototype and concurrency/guard test report as technical reference and scratch-model evidence only. S37 incorporates the 4 November catalog identity and package discussion (canonical service name immutability D01, locked package upgrade rules, admin-only MVP scope, ordinary care unblocked by debt, future reception package debt rules, and package_catalog naming).

| Source | New record | Bytes | SHA-256 |
| --- | --- | --- | --- |
| S25 | [clinic_packages_handover_2026-10-28.md](history/clinic_packages_handover_2026-10-28.md) | 40548 | 30898DA4F7B7563682FB9B6E6EFA9B91F0361754870C9FDE2553AE1DDE62265B |
| S26 | [clinic_packages_addendum_2026-10-28_to_2026-10-29.md](history/clinic_packages_addendum_2026-10-28_to_2026-10-29.md) | 37118 | 05BC1D7569CB23D54D88AE8B91BAE902D702A53813FE02EE3208F374685285F8 |
| S27 | [clinic_packages_discussion_patch_2026-11-02_181036_IST.md](history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md) | 61484 | 5CC6594D2670145F6CAE4B76A1C8FF7A769CF310A79974CE2BD4E389CAF825C6 |
| S28 | [package_decision_log_2026-11-02_voice_session.md](history/package_decision_log_2026-11-02_voice_session.md) | 15033 | 447F5CDB22CEFE78F72CD709B8CFC8BB64D3CC32DE0B08A8198FA00D6C57BE0F |
| S29 | [clinic_package_decision_log_2026-11-03_pre_questionnaire.md](history/clinic_package_decision_log_2026-11-03_pre_questionnaire.md) | 21594 | 4C046D4F06F9F98C30AA3E95A9AA230F7717D80CF4B6CDA07BDBB480E22FB79C |
| S30 | [package_module_questionnaire_reconstructed_2026-11-03.md](history/package_module_questionnaire_reconstructed_2026-11-03.md) | 51547 | F5557B48936F104DB03320DD07EF309701D237F46BB8E281661C996593C66C32 |
| S31 | [package_module_Q18-Q40_decision_addendum_2026-11-03.md](history/package_module_Q18-Q40_decision_addendum_2026-11-03.md) | 51718 | 7F39938506E8F7EA76534183382D26AB3B3BF1EF946D50C79470A7863C5438F2 |
| S32 | [package_module_final_voice_decision_addendum_2026-11-03_Q32_onwards.md](history/package_module_final_voice_decision_addendum_2026-11-03_Q32_onwards.md) | 70722 | 5676E1A64386A2FDE510D09CBC25845C37560872B0A7C0AD7765C9F883C00C0C |
| S33 | [package_call_transcript_discussion_log_2026-11-03_2231_IST.md](history/package_call_transcript_discussion_log_2026-11-03_2231_IST.md) | 11573 | E994CEF6399772334F9FF949388C2764D513C255C4587831CE5A4B256206AA95 |
| S34 | [package_upgrade_design_discussion_log_2026-11-04_0220_IST.md](history/package_upgrade_design_discussion_log_2026-11-04_0220_IST.md) | 19725 | D034827E4A7B6CEAF744D9FDFFC9DCEB8BEDB6FDFE211F34F2A949137CD361A6 |
| S35 | [response.md](history/claude/response.md) | 18466 | 23411AEF565DEBF3648B09A534D36E549809DC0954F6CC8E27ED8DED04FE5AC1 |
| S36 | [package_upgrade_prototype_and_test_results_2026-11-04.md](history/claude/package_upgrade_prototype_and_test_results_2026-11-04.md) | 59462 | 9D5CBE36B661A3E7FA1FE52E5C807ED836D1AFB93B5AB753AC9C6F1BBC7E7398 |
| S37 | [catalog_identity_discussion_2026-11-04.md](history/catalog_identity_discussion_2026-11-04.md) | 77508 | 95401D199F9B7EC471E2932C5F0242AE504D3273BDBD324872D68AF5E68507C6 |

## Decision register

The current phase has admin as its sole application role. Reception permissions describe the planned post-MVP role and its future authorization/RLS (S24, S37 §9 and §22). Clinical workflow actors do not imply extra application roles.

Rows summarize fact-level reconciliation; they are not a count of newly approved requirements. A locked business rule can coexist with an audited implementation. Original review questions Q01–Q07 are settled; historical status categories do not reopen them.

| ID | Owning topic | Current status | Current interpretation | Evidence |
| --- | --- | --- | --- | --- |
| R01 | catalog | LOCKED MVP | One operating clinic for MVP; do not implement a multi-branch matrix at launch. | S10 §2.1; S15 D-01; S16 M009 |
| R02 | catalog | LOCKED MVP | Current editable service prices and enabled/disabled availability live in services for the single clinic. Preserve issued price snapshots; do not add per-clinic price rows merely for one clinic. | S10 §2.1; S11 Part II D |
| R03 | catalog | DEFERRED FUTURE | Future sparse branch overrides are a preference, not completed architecture. Preserve explicit configured branch prices from silent global-reference updates and the requested clinic_services migration dependency. Missing-row/fallback/provisioning rules and table responsibilities remain future design. | S02 §2.1; O06 brief; S10 §2.2; S15 D-02 |
| R04 | clinical | LOCKED MVP | Ordinary therapy is flat-fee per treated complaint-visit. Two treated complaints during one physical attendance mean two visits and separate invoice lines. Ordinary billing_mode/per-procedure therapy is dropped. | S03 §§11.1–11.2; S15 D-03,D-10,D-41; S16 M148–M158 |
| R05 | clinical | LOCKED MVP | visit_services records procedures performed within a complaint-visit. Bundled routine procedures have no independent billed or zero-valued procedure lines and no separate ordinary procedure prices. | S03 §§11.1–11.2; S11 Part II D; S15 D-03,D-42 |
| R06 | catalog | EXCLUDED MVP | No persistent patient-specific negotiated/grandfathered-rate engine in MVP. Standard prices are invoiced; authorized discount settlement is handled through a waiver. | S15 D-04–D-05; S16 M009 |
| R07 | clinical | LOCKED MVP | The clinician manually assigns ROUTINE or REHAB when creating the patient complaint. Store the classification on that complaint; subsequent visits use it. Do not derive it automatically from complaint_catalog or reselect it each attendance. | S15 D-39; S16 M118–M131 |
| R08 | clinical | LOCKED WITH PARKED LIFECYCLE | Classification is permanent. A different classification requires a new complaint while preserving the original and its history. No mandatory/automatic closure of the old complaint is approved. | S15 D-40; S16 M134–M145,M171–M174 |
| R09 | catalog | LOCKED MVP | Use one standard routine flat rate and one standard rehabilitation flat rate, independent of the ordinary procedures performed. The later transcript does not choose new numerical rate values or rate-table DDL. | S15 D-41; S16 M009,M148–M158 |
| R10 | catalog | LOCKED MVP / Q07 RESOLVED | Routine and standalone service identities come from services; no free-text service identity. Only admin may add services, edit existing master catalog prices or enable/disable services. Reception cannot change master prices or availability. Backend enforcement must accompany UI restrictions. For new entries, select only enabled services; backend independently rejects unavailable service IDs. Concurrent status changes remain audited. Canonical service names are immutable once visits are recorded (D01; S37). | S20; S15 D-42–D-43; S16 M163–M168; S02 §1; S37 D01 |
| R11 | clinical | LOCKED DIRECTION / AUDIT REQUIRED | Every complaint course references complaint_catalog; no uncataloged free-text complaint identity. Derive complaint anatomy through that catalog in the discussed model; shared region representation/migration remains audited. Narrative notes are not prohibited. | S15 D-30; S16 M030–M034 |
| R12 | clinical | LOCKED MVP | Admin may add a missing complaint-catalog entry before registering the patient's complaint. Those additions require an append-only who/when/what trail; generic catalog versioning or system-wide audit DDL was not chosen. | S15 D-31; S16 M033–M036 |
| R13 | exams | EXCLUDED MVP | No appointment/scheduling module is introduced. Eligibility refers to qualifying completed attendance, including fee-only attendance, not bookings, cancellations, or no-shows. | S11 Part I §9 and Part II A; S23 |
| R14 | exams | LOCKED MVP | Creating a new complaint or performing routine clinical assessment does not itself trigger an examination fee during ongoing care. Examination/lapse fees are billing terms under the first-attendance/return-gap rule, not records of clinical assessment. No new-complaint examination-fee trigger is approved. | S11 Part I §§6–7; S23 |
| R15 | exams | LOCKED BUSINESS / AUDIT REQUIRED | An applicable examination fee can be checked out without therapy, for both first attendance and a qualifying recurrent return under the same rules. Therapy may start on another day. No complaint-visit or visit_services record is required or fabricated for the fee. One charge per physical attendance must not multiply with complaint-visits; exact enforcement remains audited. | S11 Part I §§1–3,8; S23 |
| R16 | exams | LOCKED BUSINESS / AUDIT REQUIRED | Use calendar-day inactivity since the patient's most recent qualifying completed attendance, not last payment or last formal exam. Therapy-only and fee-only attendance both reset recency. A first fee-only checkout followed by next-day therapy therefore incurs no new gap fee. Exact record mapping, other encounter types and completion remain audited. | S11 Part I §§4–5 and Part II A; S15 D-26; S16 M011,M073,M098; S23 |
| R17 | exams | LOCKED MVP | First qualifying attendance: ₹350 (35000 paise). Return gap 0–9 calendar days, including five days: no examination/lapse fee event; 10–29: ₹150 (15000 paise); 30+: ₹350 (35000 paise). Never stack ₹150 and ₹350. Late fee means return-gap charge, not late arrival/payment. Routine assessment does not itself trigger a fee. | S11 Part II B; S15 D-06,D-34; S16 M048–M050,M068–M073,M098; S23 |
| R18 | exams | CLARIFIED MVP | Examination fee and late/lapse fee are billing terms, not proof that a clinical examination occurred or how thorough it was. Ongoing clinical assessment is distinct. Backend Create Invoice automatically applies the applicable fee without receptionist examination confirmation or a separately recorded clinical examination prerequisite. | S17 U03; S11 Part I §4; S15 D-35; S16 M089–M099; S23 |
| R19 | exams | LOCKED DIRECTION | Keep actual standalone treatment occurrences separate. Combined extra_fees represents independently recordable patient-level examination/return-gap charge events referenced by invoice lines. A fee does not require a complaint-specific visit_id or visit_services row and is not proof of a performed examination. Exact fee schema remains audited. | S15 D-33; S16 M064,M066,M098–M099; S23 |
| R20 | exams | LOCKED BUSINESS / AUDIT REQUIRED | Backend fee eligibility, applicable fee records, immutable calculation evidence, and invoice lines are handled before issuance inside atomic checkout, including fee-only checkout. The frontend cannot insert arbitrary fee events. Exact snapshot fields and concurrent duplicate prevention remain audited. | S15 D-35; S16 M073,M089–M099; S23 |
| R21 | checkout | LOCKED MVP / Q05 RESOLVED | Before Create Invoice, review what was done today for clinical accuracy. The preview shows clinical records, without automatic fee amounts or invoice prices/totals. Existing last-visit context may remain visible. After successful atomic issuance, display the complete itemized invoice with automatic fees and total; settlement follows separately. Backend calculates applicable fees before commit inside atomic issuance,not after the invoice is issued. | S19; S15 D-35; S16 M089–M099 |
| R22 | checkout | LOCKED MVP | MVP issuance atomically saves that checkout's confirmed performed-care records when present, applicable extra-fee events, invoice and lines: all commit or all roll back. Fee-only checkout requires no fabricated clinical record. No clinical-save-first/bill-later path is approved. Settlement is separate after the invoice is returned. | S17 U04 corrects S12 Q3; S13 A8; S15 D-15; S16 M009,M089; S23 |
| R23 | checkout | LOCKED MVP | No clinical-save-first/bill-later MVP path is permitted. That checkout's confirmed performed-care records when present, applicable fees, invoice and lines commit atomically; fee-only issuance remains valid without dummy care. Possible future invoice-line work does not authorize partial clinical persistence. | S17 U04; S18; supersedes S12 Q3 exception; S23 |
| R24 | payments | LOCKED MVP | Issuance and settlement are separate transactions. Create and return the invoice first; only afterward record money and/or an authorized final waiver. Waiving during initial checkout means the initial visit's later settlement stage, not payment writes in issuance. | S13 A8; S14 D26-09 and §8; S15 D-22,D-36; S16 M099–M106 |
| R25 | checkout | LOCKED MVP | Complaint therapy, standalone care delivered with it, and an applicable examination/return fee appear as distinct lines on the same checkout invoice. One standalone line represents one actual performed occurrence; individual standalone sessions are not presold. | S11 Part I §2; S15 D-08,D-11,D-12; S16 M009,M163–M165 |
| R26 | clinical | LOCKED DIRECTION | Each independently performed standalone treatment is its own timestamped immutable patient-linked occurrence and its own billable source, without a fabricated complaint-visit. Two actual performances remain distinct even for the same catalog service/body region. | S03 §11.5; S15 D-08–D-09; S16 M009 |
| R27 | clinical | LOCKED MVP | Do not link standalone treatment occurrences to complaint_course_id in the latest discussed model. This supersedes the optional-link recommendation and its local complaint-ownership constraint; it does not cancel the schema-wide ownership audit. | S15 D-28; S16 M020–M023 |
| R28 | clinical | LOCKED REQUIREMENT / AUDIT REQUIRED | Every standalone treatment must identify its anatomical body region. Shared body_regions lookup versus other representation and cross-catalog migration remain audited; the assistant's table-versus-enum declaration is not independent approval of storage. | S15 D-29; S16 M026–M031 |
| R29 | clinical | LOCKED MVP / Q06 RESOLVED | Standalone clinical occurrences store mandatory service_id and display the catalog name through a join; no extra clinical treatment-name snapshot. Invoice-line descriptions and amounts remain immutable snapshots. Common server recorder/timestamp conventions remain settled; exact DDL remains audited. | S21; S16 M006,M008; S15 D-32 |
| R30 | clinical | LOCKED MVP | No physical deletion or retroactive correction/voiding of performed standalone/clinical history is approved for MVP. Preserve recorded events and issued charges; absence of a correction feature does not make mistaken entries truthful. | S09 §§1–2; S15 D-09,D-23; S16 M009 |
| R31 | checkout | LOCKED MVP | Issued invoice line descriptions and amounts are immutable historical snapshots, including while unpaid/partially paid. Sum issued lines for billed total; changing catalog prices cannot rewrite existing bills. | S03 §11.4; S12 Q6 and §3.2; S14 D26-02; S15 D-16 |
| R32 | checkout | LOCKED MVP / Q01 RESOLVED | At least one invoice line is mandatory. Zero-total invoices are permitted in exactly two validated cases: (1) package-covered-only visits validated by backend (Q01; S17 U01), and (2) calculated-zero package upgrades where current long price <= original invoiced charge (S37 §25). Other zero-total invoices, arbitrary free package sales, and extra zero-value bundled-procedure lines are not authorized. | S17 U01; S14 D26-03,D26-08; S03 §11.3; S37 §25 |
| R33 | checkout | LOCKED EXCEPTION / AUDIT REQUIRED | Covered complaint-visits retain zero-valued invoice lines, including an entirely covered zero-total invoice. Ordinary bundled procedures receive no extra zero-value lines. Package eligibility/redemption implementation and broader line rules remain audited. | S17 U01; S03 §§11.1–11.3; S15 §5.2 |
| R34 | checkout | PROVISIONAL DEFAULT / AUDIT REQUIRED | Typed real source FKs are the working default; require exactly one logical billable source per line. Context FKs are not additional billable sources. An extra_fees source does not require a clinical examination or complaint-visit source. Database enforcement is required, but exact columns/CHECK are not finalized. | S03 §§5–7; S15 D-13,D-17,D-33; S16 M009,M098; S23 |
| R35 | checkout | LOCKED REQUIREMENT / AUDIT REQUIRED | Prevent billing the same source occurrence twice across invoice lines/invoices. Uniqueness belongs to an actual visit/treatment/fee occurrence, never the reusable catalog service_id. One visit belongs to at most one invoice; no one-invoice-per-day rule or final membership FK is approved. | S03 §§5–7; S15 D-14,D-17; S16 M009 |
| R36 | payments | LOCKED MVP | PAID means the billed balance is discharged by actual receipts or through the explicitly permitted zero-total settlements (package-covered-only visits, or calculated-zero package upgrades where current long price <= original invoiced charge; S17 U01, S37 §25). WAIVED means admin forgave the entire residual, possibly after partial receipts. Both are terminal with zero collectible balance; no later payment, waiver or state mutation. | S17 U01; S10 §1; S12 Q2,Q5–Q6; S15 D-23; S37 §25 |
| R37 | payments | LOCKED MVP | Four states are UNPAID (issued,no receipt),PARTIALLY_PAID (positive receipts with collectible balance),PAID and WAIVED. The user explicitly approves this naming set in S21. Terminal behavior and the permitted zero-total PAID exceptions (package-covered-only visits and calculated-zero package upgrades) remain unchanged. | S21 supersedes S18 terminology; S17 U01; S12 §3.3; S37 §25 |
| R38 | payments | LOCKED MVP | The current delivery phase uses admin only. When the planned reception role is introduced post-MVP, it may finish initial collection after a browser interruption, checking the stored outcome before another write. For positive invoices: Pay Later/no receipt leaves UNPAID, a partial positive receipt leaves PARTIALLY_PAID, and full collection closes PAID. Pay Later ends that initial interaction; later receipts remain allowed under R39. The permitted zero-total invoices (covered-only visits and calculated-zero upgrades) retain only Mark as paid, with no payment row. In a planned post-MVP reception phase, receptionists are blocked from selling packages if any patient package-linked invoice is unsettled, while ordinary completed care is never blocked by patient debt (S37). | S17 U01–U02; S12 Idea B and §3.4; S21 terminology; S24; S37 |
| R39 | payments | LOCKED MVP | Admin now, and the planned reception role when introduced, may record repeated positive receipts against UNPAID or PARTIALLY_PAID invoices with collectible balance, including after Pay Later and on later days. There is no initial-checkout-only, invoice-age or arbitrary repayment-count limit. Partial collection leaves PARTIALLY_PAID; full balance collection closes PAID. Reception need not hand those receipts to admin. Wallets and automatic cross-invoice allocation remain excluded. | S12 Q1 and Idea C; S14 D26-04; S21 terminology; S24 |
| R40 | payments | LOCKED MVP | Only admin/owner may waive; the planned reception role must have neither waiver controls nor backend waiver permission. Permitting initial/later receipts does not permit modifying issued lines, saved payments or clinical history. The current phase has admin as the sole application role; reception-specific role/authorization/RLS is planned for post-MVP (reception is the first planned post-MVP addition; S37 §9, §22). Exact enforcement remains audited. | S12 Q1–Q2 and Idea B; S15 D-20; S24; S37 §9, §22 |
| R41 | payments | LOCKED RULE / AUDIT REQUIRED | Requested total collection cannot exceed current collectible outstanding. Enforce on frontend and backend under concurrency; exact implementation/locking is not completed by the source set. Reject settlement attempts on terminal invoices. | S13 A5; S15 D-23 and §§5–6 |
| R42 | payments | LOCKED MVP | Backend payment methods are an enum limited to Cash and UPI. Split is a frontend input mode, not a third stored method. Card and bank transfer are excluded; exact SQL naming/casing is not selected by this record. | S13 A3 |
| R43 | payments | LOCKED MVP | Split entry explicitly accepts both Cash and UPI amounts. Sum them and calculate the residual; do not infer full payment or autofill the second component from the full invoice total. | S13 A4 |
| R44 | payments | LOCKED MVP | Each positive tender component is a separate payment row. Insert all split components and update invoice settlement status in one payment transaction; any failure rolls back the whole settlement. No zero components become payment rows. | S12 §3.1; S14 D26-05,D26-15 |
| R45 | payments | LOCKED MVP | Each payment row belongs to exactly one invoice in MVP; an invoice may have zero, one, or many actual receipts. No unified receipt-allocation engine or automatic cross-invoice distribution is approved. | S14 D26-04; S12 Idea C |
| R46 | payments | LOCKED MVP | Keep created_at and received_at as server-controlled TIMESTAMPTZ NOT NULL with server defaults. No client backdating. Delayed admin entry reports on the recording day; this accepted limitation supersedes the older absolute same-day-entry interpretation. Future distinct receipt time is not current backdating support. | S13 A6; S14 D26-06,D26-10 |
| R47 | payments | LOCKED DESIGN / AUDIT REQUIRED | Payment recorded_by is required and derived from authenticated backend identity, not trusted from React. User baseline references profiles(id) UUID, but physical FK/default/security integration still requires schema review. | S14 D26-12; S16 M009 |
| R48 | payments | LOCKED MVP | Use BIGINT integer paise for billing/payment monetary values. Persist payments only for positive actual money. Package-covered zero invoice lines and zero-total settlement do not create a zero payment or fictitious waiver receipt. | S17 U01; S14 D26-05,D26-11,D26-13 |
| R49 | payments | LOCKED MVP | Successfully saved payment rows cannot be edited, deleted, reversed, or retrospectively corrected in MVP, including while the invoice remains nonterminal. Verify entered amount before submission. | S12 §3.4; S14 D26-13 |
| R50 | payments | LOCKED MVP | Invoice deletion is forbidden on frontend and backend; payment-to-invoice FKs use ON DELETE RESTRICT. Preservation is not permission to edit terminal settlement or issued lines. | S14 D26-14 |
| R51 | payments | LOCKED MVP | Use invoice-level final-waiver metadata, no redundant is_waived boolean and no separate invoice_waivers table for this one-final-waiver model. Waiver attaches to the entire invoice's residual rather than particular lines. | S10 §1.3; S13 A2; S15 D-21 |
| R52 | payments | LOCKED MVP | WAIVED requires a nonempty free-text reason; frontend text presets are allowed. Ordinary full payment requires no reason. Waiver/write-off share WAIVED without separate WRITE_OFF status or structured reason enum. Reason examples do not authorize a correction workflow. | S13 A1,A9; S14 D26-07 |
| R53 | payments | LOCKED DESIGN | Use settled_at and settled_by for final PAID or WAIVED closure instead of separate paid_at/waived_at/waived_by. waiver_reason is required for WAIVED and null for PAID. Precise nonterminal constraints and physical migration remain audited. | S14 D26-09 and §6 |
| R54 | payments | LOCKED DIRECTION / AUDIT REQUIRED | Derive billed from immutable lines and received from positive payments. For terminal WAIVED, forgiven=billed−received and collectible outstanding=0; otherwise forgiven=0 and outstanding=billed−received. No redundant authoritative total, waived_amount, or outstanding_amount columns. Query/view and aggregation safety remain audited. | S10 §1.2; S14 D26-01–D26-02,D26-07; S15 §5 |
| R55 | payments | LOCKED MVP | Final waiver forgives the entire remainder and ends WAIVED, whether earlier receipts exist or zero money was received. Partial waiver while keeping the invoice PARTIALLY_PAID is outside this MVP model. A full waiver creates no payment row. | S10 §1.1; S12 Q2,Q5; S14 D26-01; S21 terminology |
| R56 | payments | LOCKED MVP | Discounts use standard issued prices, actual positive receipt(s), and admin waiver of the remaining collectible balance. This leaves WAIVED; it does not reprice lines or approve an overcharge/clinical-error correction feature. | S15 D-04–D-05,D-21–D-22; S09 §§1–2 |
| R57 | payments | LOCKED MVP | Collection reports sum actual dated positive receipts, including receipts on still-PARTIALLY_PAID and later-WAIVED invoices. Do not use billed totals, PAID-only invoice filters, or forgiven amounts as cash. Delayed-entry reporting follows R46. | S12 §3.4; S14 D26-06; S21 terminology |
| R58 | payments | LOCKED MVP | No separate numbered/printed/sent payment receipt artifact is required. Keep the backend payment record and clear saved-amount/method/status/balance confirmation. | S12 §3.4; S09 §6 |
| R59 | payments | LOCKED BUSINESS / AUDIT REQUIRED | Show payment success only after confirmed backend save. Failed or uncertain responses cannot claim success. Exact API payloads, UI state, cancellation, and error/recovery contracts remain audited. | S12 §3.4; S15 §6 |
| R60 | checkout | LOCKED REQUIREMENT | Disable submit while a request is pending. Preserve the form and allow manual retry after confirmed rollback; no automatic client/server retry loop. Backend duplicate prevention is still mandatory; button disabling and atomicity do not prove idempotency. | S12 Q4; S15 D-18–D-19; S16 M009 |
| R61 | checkout | PROVISIONAL / AUDIT REQUIRED | The proposed idempotency-key/request-hash/invoice record is not final. Audit request scope, canonical hashing, concurrent races, retention, lost-response recovery, and payment replay. Unknown outcome is not confirmed rollback. | S15 D-24; S16 M108–M109 |
| R62 | packages | LOCKED BASE MVP / CONDITIONAL UPGRADE | MVP includes ordinary 10- and 26-visit offerings, admin-only operation, payment-independent coverage from the valid package start date (T or T+1), default daily cap 1, strict closure irreversibility, immutable submitted justification notes, six active closure choices including Other; "Duplicate Package Purchased" is deferred; the "Patient Stopped Attending" date gate remains, no advance queue, combined checkout, and package_catalog/packages naming. The 10→26 upgrade design is settled: same complaint and category; terminal 10-visit parent (Completed or Closed on/after its own review date); purchase by the original inclusive Day 30; no prior upgrade or intervening replacement; +16 visits; current 26-visit catalog price minus the original 10-visit purchase-line invoiced charge, floored at zero; original start date governs the Day-30 upgrade review; copy the parent daily cap; show invoice-level refunds without blocking; issue calculated-zero invoices before separate Mark as paid. Shipping upgrades in MVP remains conditional on bounded engineering effort (S34 §6, A11/N51). | S25–S37; S34 §6; S37 §16, §21–§25 |
| R63 | checkout | DEFERRED FUTURE | No post-issuance append/remove/reprice, generic replacement/void-and-reissue, saved-payment repair or retroactive clinical correction in MVP. No unused voiding fields are preapproved. (The manual administrative workaround for unpaid mixed mistaken-package invoices—terminally waiving the whole erroneous invoice and creating a clean replacement invoice for legitimate charges under S31 Q21—is an authorized manual financial procedure, not an automated invoice mutation or generic void tool). | S09 §§1–2,7; S12 Q6; S31 Q21 |
| R64 | cross | CONDITIONAL FUTURE | If correction/voiding is later built, preserve conditional admin-only authority, child and parent consistency, Paid-link blocks, financial floors, and audit metadata. Waiver-plus-void double reduction and false-record/Paid-lock traps remain unresolved future risks. | S07 §§1–6; S08 G06–G15; S09 §§1–2 |
| R65 | cross | AUDIT REQUIRED | Retain historical dispositions for vitals (drop/dormant unresolved), alerts (removal intent subject to dependency/data checks), and timeline_events (purpose to investigate). No cleanup, deletion, or live-schema inventory was performed here. | S06 §5; S08 §4; existing clinical topic |
| R66 | cross | AUDIT REQUIRED | Legacy daily_ledger Complete/Pending/Due semantics and current collection derivations need review. Avoid multiplying line/payment sums through a naive join. A documented view or proposed check is not implemented evidence. | S06 §5; S08 §4; S14 §6 |
| R67 | cross | AUDIT REQUIRED | Review schema-wide patient/clinic/complaint reference necessity, normalization versus stored ownership, composite constraints, and handling missing/contradictory client IDs. Removing a standalone complaint link does not settle this system-wide question. | S15 §6 and §8.2; S16 M008,M020–M023 |
| R68 | clinical | AUDIT REQUIRED | Review shared anatomy representation across complaint_catalog, complaint courses, and standalone treatments; do not automatically add duplicate course anatomy or assert current schema from assistant reports. | S15 D-29–D-30 and §6; S16 M030–M033 |
| R69 | exams | AUDIT REQUIRED | Billing-only meaning and independence from complaint-specific visits are settled. Audit exact extra_fees representation, fee-only attendance/recency mapping and source ownership without requiring visit_id, visit_services or a performed-examination record for charging. This fee policy does not select a clinical examination table. | S17 U03; S11 Part I §8; S15 D-27; S16 M001–M002,M014; S23 |
| R70 | clinical | PARKED AUDIT | Complaint closure/inactivity, indefinite active state, and what happens to an old complaint after a replacement are explicitly for audit. No auto-close or mandatory close-before-replacement rule is chosen. | S15 D-40; S16 M144–M145,M171–M174 |
| R71 | cross | CLOSED SESSION WITH EXCLUSIONS | The final transcript closes the active non-audit business discussion, after excluding audits and packages. It does not establish completed schema, SQL/API/RLS, migration, or executed acceptance/concurrency checks. | S15 D-38,D-44 and §§4,6,9; S16 M114,M169–M176 |
| R72 | cross | AUDIT REQUIRED | Keep exact DDL, indexes, RPCs, authorization/ownership, concurrency, financial queries, migration/rollback and validation as unfinished engineering follow-up. Proposed payment indexes are not approved merely because they were suggested. | S14 §6 and §8; S15 §§4–6; existing O13–O18 |
| R73 | cross | PROCESS / EVIDENCE LIMIT | Preserve source records; discuss one meaningful unresolved choice at a time and distinguish user approval from assistant proposal. The export's displaced opening approvals and Show more markers limit exact chronology. Documentation consolidation is not schema implementation. | S14 §0,§7; S15 D-25 and metadata; S16 M001–M009 |
| R74 | checkout | USER RESOLUTION Q01 | Package-covered-only zero invoices are allowed, with at least one real line and backend-validated coverage. After issuance offer only Mark as paid, close PAID with existing settlement metadata, create no payment row and report zero collections. Positive uncovered fees use ordinary settlement. S37 §25 adds eligible calculated-zero upgrade invoices to the same separate settlement flow (R32, R36, R38). | S17 U01; S22 Q01; S37 §25 |
| R75 | payments | SUPERSEDED LABELS / CURRENT NAMES APPROVED | The earlier choice to retain PENDING/DUE labels is superseded by the explicit S21 selection of UNPAID/PARTIALLY_PAID. These are the same four status meanings,with PAID and WAIVED unchanged. Historical sources retain their original wording. | S21 supersedes S18 label choice; S17 U02 |
| R76 | exams | USER CLARIFICATION Q03 | Backend Create Invoice checks patient-wide qualifying attendance and calendar-day gap, and automatically includes any applicable fee in the returned invoice. First and recurrent fee-only checkout follow the same rules. No clinical examination record or manual examination-confirmation gate is required. Fees first appear after issuance. | S17 U03; S15 D-35; S16 M089–M099; S22 Q03; S23 |
| R77 | checkout | USER CORRECTION Q04 | No clinical-save-first/bill-later path in MVP: that checkout's confirmed performed-care records when present, applicable fees, invoice and lines commit atomically. Fee-only issuance needs no dummy clinical record. S12 Q3's earlier exception is a source-summary error. Payment/waiver settlement follows separately after the invoice is returned. | S17 U04; S12 Q3 line 52; S22 Q04; S23 |
| R78 | cross | CURRENT DECISION AUTHORITY | Settled current rules agree across topic pages,register and tracker current columns. Historical evidence cannot override them. All seven review questions are settled,including new status names and service_id-only clinical identity; parked implementation audits remain open. | S18–S21 |
| R79 | checkout | USER RESOLUTION Q05 | Before Create Invoice, review what was done today for clinical accuracy. The preview shows clinical records, without automatic fee amounts or invoice prices/totals. Existing last-visit context may remain visible. After successful atomic issuance, display the complete itemized invoice with automatic fees and total; settlement follows separately. Backend fee calculation remains inside the atomic issuance transaction. | S19; S22 Q05 |
| R80 | catalog | USER RESOLUTION Q07 | Only admin may edit existing master catalog prices and enable/disable services. Reception cannot change them. Admin-only additions remain settled; backend authorization and UI restrictions must enforce this authority. | S20; S22 Q07 |
| R81 | payments | USER APPROVED TERMINOLOGY | Use UNPAID,PARTIALLY_PAID,PAID,WAIVED. PENDING maps to UNPAID and DUE maps to PARTIALLY_PAID; behavior, terminal states and permitted zero-total settlements remain unchanged. This is an explicit later rename, not a new state or completed enum migration. | S21; S22 Q02; S37 §25 |
| R82 | clinical | USER RESOLUTION Q06 | Standalone clinical occurrence stores service_id and displays the catalog name through a join,without an extra clinical name snapshot. Invoice-line snapshots remain required. Aliases for alternate wording are catalog metadata direction; exact alias representation is not selected. | S21; S22 Q06 |

## Source coverage

S25–S37 package and catalog records are carried into the [package decisions page](packages.md), related topic pages, and A11/N26/N32/N51/R62. Package lifecycle, upgrade economics, admin-only MVP scope, and canonical service name immutability are fully reconciled through 5 November 2026.

### Five incoming addenda

| Source units | Reconciled coverage |
| --- | --- |
| S10 §§1.1–1.3 | R36,R51–R55; waiver-as-Paid is superseded, not retained as a parallel current outcome |
| S10 §§2.1–2.2 | R01–R03; single-clinic launch versus future clinic_services migration |
| S10 §§3–4 | Scope/provenance boundaries carried through the register, not new DDL approvals |
| S11 Part I §§1–3 | R15,R25; exam-only attendance, shared checkout, single charge per attendance |
| S11 Part I §§4–5 | R16–R18; calendar recency; Q03 clarified by S17 U03 as clinical protocol versus automatic charging |
| S11 Part I §§6–10 | R13–R15,R62,R69,R71; new-complaint trigger excluded, modeling open, earlier checkpoint qualified |
| S11 Part II A–B | R16–R17; calendar dates and precise fee bands closed |
| S11 Part II C–F and checkpoint | R01–R06,R25–R35,R62,R72; no reopened billing_mode, no new package-line ruling |
| S12 Q1–Q6 | R22–R24,R36–R40,R55,R60,R63; earlier clinician-save exception corrected by S17 U04; source preserved |
| S12 Ideas A–C | R38–R39,R45,R51; is_waived unanswered there but later rejected in S13 |
| S12 §§3.1–3.4 | R36–R61; receipts, derived amounts, states, reporting and feedback |
| S12 §§4–5 | Outcome approval does not approve every FK/index; later sources narrow earlier parked cardinality questions |
| S13 A1–A9 and B Q1–Q15 | R24,R41–R46,R51–R56; methods, split, flags, reasons and transaction boundary; payment cardinality later closed by S14 |
| S13 C–E | R61–R63,R72; incomplete technical work, explicit parking, and historical handoff preserved |
| S14 initial D26-01–D26-09 and later D26-10–D26-15 | Every numbered decision mapped below; appended payment fields must not be lost behind the earlier checkpoint |
| S14 §§2–7 and indexes proposal | R32,R46,R52–R54,R71–R73; audit/counted-idea corrections, fixed roadmap and parked topics preserved; suggested indexes remain open |

| Running-log ID | Register rows |
| --- | --- |
| S14:D26-01 | R54,R55 |
| S14:D26-02 | R31,R54 |
| S14:D26-03 | R32 |
| S14:D26-04 | R45 |
| S14:D26-05 | R44,R48 |
| S14:D26-06 | R46,R57 |
| S14:D26-07 | R52 |
| S14:D26-08 | R32,R33 |
| S14:D26-09 | R53 |
| S14:D26-10 | R46 |
| S14:D26-11 | R48 |
| S14:D26-12 | R47 |
| S14:D26-13 | R48,R49 |
| S14:D26-14 | R50 |
| S14:D26-15 | R44 |

### Updated MVP register

| MVP log ID | Register rows |
| --- | --- |
| S15:D-01 | R01 |
| S15:D-02 | R03 |
| S15:D-03 | R04,R05 |
| S15:D-04 | R06 |
| S15:D-05 | R56 |
| S15:D-06 | R17 |
| S15:D-07 | R62 |
| S15:D-08 | R25,R26 |
| S15:D-09 | R26,R30 |
| S15:D-10 | R04 |
| S15:D-11 | R25 |
| S15:D-12 | R25 |
| S15:D-13 | R34 |
| S15:D-14 | R35 |
| S15:D-15 | R22,R23,R24 |
| S15:D-16 | R31 |
| S15:D-17 | R34,R35 |
| S15:D-18 | R60 |
| S15:D-19 | R60 |
| S15:D-20 | R40 |
| S15:D-21 | R51 |
| S15:D-22 | R24,R56 |
| S15:D-23 | R36 |
| S15:D-24 | R61 |
| S15:D-25 | R73 |
| S15:D-26 | R16 |
| S15:D-27 | R69 |
| S15:D-28 | R27 |
| S15:D-29 | R28 |
| S15:D-30 | R11 |
| S15:D-31 | R12 |
| S15:D-32 | R29,R47 |
| S15:D-33 | R19 |
| S15:D-34 | R17 |
| S15:D-35 | R20,R21 |
| S15:D-36 | R24 |
| S15:D-37 | R37 |
| S15:D-38 | R71 |
| S15:D-39 | R07 |
| S15:D-40 | R08,R70 |
| S15:D-41 | R04,R09 |
| S15:D-42 | R10 |
| S15:D-43 | R10 |
| S15:D-44 | R71 |

### Entire JSON export

| Messages (one-based) | Exchange / source boundary | Register coverage |
| --- | --- | --- |
| M001–M008 | Displaced exam/normalization audit requests and short approvals | R29,R67,R69 |
| M009 | User-supplied baseline; not proof of implementation | R01–R73, qualified by later changes |
| M010–M014 | Any completed encounter and exam modeling audit | R16,R69 |
| M015–M023 | Standalone proposals, removed complaint link, derived total | R27,R29,R31 |
| M024–M030 | Mandatory anatomy and normalization discussion | R28,R68 |
| M031–M036 | Catalog-backed complaints, admin additions and addition audit | R11,R12 |
| M037–M065 | Fee/examination alternatives and combined fee direction | R18,R19,R34 |
| M066–M098 | Fee bands, backend calculation, issued invoice visibility, user lock | R17,R20,R21 |
| M099–M109 | Later payment transaction, status terminology, idempotency audit | R24,R37,R61 |
| M110–M114 | Remaining audit inventory excluded from active session | R69,R71,R72 |
| M115–M131 | Manual permanent complaint classification assignment | R07 |
| M132–M145 | No reclassification; replacement and closure parked | R08,R70 |
| M146–M158 | Two flat prices, per-complaint-visit billing | R04,R09 |
| M159–M168 | Catalog-only services, admin additions, approval boundary | R10 |
| M169–M176 | Session completion and final lifecycle audit parking | R70,R71 |

All 176 positions are represented once in this range map; acknowledgement fragments do not become new approvals. S15's earlier reconstructed exchanges remain carried-forward history where the export does not reproduce them.

## Resolved overrides and clarifications

| Earlier position | Evidence-backed treatment | Register |
| --- | --- | --- |
| Waiver ends Paid | Final WAIVED, including partial receipts plus forgiven remainder; receipts remain real payments | R36,R55 |
| Waiver flag still unanswered | is_waived explicitly dropped; invoice status/metadata suffice | R51 |
| Separate paid_at/waived_at/waived_by; all admin settlements require notes | Unified settled_at/settled_by; waiver_reason only mandatory on WAIVED, null on PAID | R52,R53 |
| Stored billed/waived/outstanding header totals | Derive from immutable lines, real payments and terminal-waiver classification | R31,R54 |
| Only PAID invoice totals count as collections | Use actual positive dated receipts, including PARTIALLY_PAID and later-WAIVED invoices | R57 |
| Absolute same-day recording; missed receipts cannot be recorded later | Later user accepts server-stamped entry-day reporting for delayed admin entry; no client backdating | R46 |
| No separate created_at/received_at needed | Both kept, server-controlled and non-null, for distinct conceptual timestamps | R46 |
| Cash/UPI/card/bank or split implies full payment | Cash/UPI only; Split is UI mode; both component amounts explicit; partial split allowed | R42–R44 |
| One-payment/one-invoice question unanswered | S14 explicitly chooses exactly one invoice per row for MVP | R45 |
| Multibranch configuration is a launch dependency | One-clinic services pricing/availability now; future sparse overrides only preferred; preserve clinic_services migration dependency | R01–R03 |
| Ordinary per-procedure billing and billing_mode | Flat complaint-visit fee; routine procedures are clinical records only | R04,R05 |
| Fees ambiguous, day-31/exact-hour question open, exam-only hypothetical | Calendar 0–9 / 10–29 / 30+ bands and exam-only attendance are explicitly decided | R15–R17 |
| New complaint triggers formal exam, negotiated rate, or per-visit tier selection | No automatic new-complaint exam; no negotiated-rate engine; manually set permanent complaint classification | R06–R09,R14 |
| Optional standalone complaint link and optional anatomy | No complaint-course link; anatomy is mandatory, storage representation remains audited | R27,R28 |
| Every fee needs a distinct table or universal billing-event layer | Combined extra_fees charge source; clinical examinations still audited; universal proposals not adopted | R19,R69 |
| Source uniqueness on service catalog identity | Once per actual occurrence, allowing separate performances of the same service | R35 |
| Invoice, clinical work and payment all in one transaction | All MVP clinical/billing issuance is atomic; no clinical-first exception (S17 U04); settlement is a distinct later transaction | R22,R24 |
| Append/void/correction permitted on Pending/Due | Intervening permissions explicitly withdrawn from MVP, historical future safeguards preserved | R63,R64 |
| Optional standalone separate invoice | Standalone and complaint therapy in the same checkout share one invoice with separate lines | R25 |
| All original design blocks are complete | Only the narrowed non-audit business discussion closes; audits, packages and implementation remain unfinished | R71,R72 |

## Questions for user review

Q01–Q04 are resolved/corrected in S17 and Q05 is resolved in S19. Q07 is resolved in S20. S21 settles Q06 and explicitly renames the nonterminal states. All seven review questions are settled; technical audits remain separate. Original question IDs are retained for traceability.

### Q01 — Covered-only invoices

- **Classification:** Former contradictory requirements, now resolved.
- **Evidence:** S03 §11.3; S14 D26-08; S17 U01.
- **Answer:** A package-covered-only invoice may total zero, with at least one real line and validated coverage. After issuance, offer only Mark as paid; PAID closure uses settlement metadata, no payment row and zero collections. Positive uncovered amounts follow ordinary settlement.
- **Current disposition:** CLOSED. The covered-only exception was settled here. S37 §25 subsequently added eligible calculated-zero upgrades under the same separate settlement flow (R32, R36, R38, R74).

### Q02 — Invoice status names — settled

- **Classification:** Explicitly approved later rename.
- **Evidence:** S21,user selects the suggested set.
- **Answer:** UNPAID,PARTIALLY_PAID,PAID,WAIVED. UNPAID means no receipt; PARTIALLY_PAID means positive receipts with a remaining collectible balance. PAID and WAIVED retain their terminal meanings. The new names supersede the earlier PENDING/DUE labels; business behavior is unchanged.
- **Current disposition:** CLOSED MVP (R37,R75,R81). This is four states; no database enum migration is performed.

### Q03 — Automatic billing fees and clinical assessment

- **Classification:** Scope clarification, not a competing billing rule.
- **Evidence:** S11 Part I §4; S15 D-35; S16 M089–M099; S17 U03.
- **Answer:** Examination fee and late/lapse fee are billing terms, not proof that a clinical examination occurred or how thorough it was. Ongoing clinical assessment is distinct. Backend Create Invoice automatically applies the applicable fee without receptionist examination confirmation or a separately recorded clinical examination prerequisite. First and recurrent fee-only checkout follow the same rules, independently of complaint visits (S23).
- **Current disposition:** CLOSED CLARIFICATION (R18,R20,R69,R76). Q05 below settles display timing.

### Q04 — Clinical recording before billing

- **Classification:** Corrected source-summary error.
- **Evidence:** S12 Q3 line 52 expressly permits clinical saving before later billing. The user explicitly rejects that and says it was never agreed (S17 U04).
- **Answer:** Performed-care records when present, applicable fee events, invoice and lines commit atomically in MVP. Fee-only checkout needs no dummy care (S23). Payment/waiver is separate after the issued invoice is returned. Possible future invoice-line work is not a clinical-first persistence path.
- **Current disposition:** CORRECTED (R22–R23,R77). Preserve S12 unchanged while superseding its erroneous exception.

### Q05 — Automatic fee visibility — settled

- **Classification:** Resolved UX behavior.
- **Evidence:** S19; S15 D-35; S16 M089–M099.
- **Answer:** Before Create Invoice, review what was done today for clinical accuracy. The preview shows clinical records, without automatic fee amounts or invoice prices/totals. Existing last-visit context may remain visible. After successful atomic issuance, display the complete itemized invoice with automatic fees and total; settlement follows separately. Fee calculation remains inside atomic issuance. This supersedes the earlier requirement for a financial preview before issuance.
- **Current disposition:** CLOSED MVP (R21,R79). Q06 is settled by S21; Q07 is settled by S20.

### Q06 — Standalone clinical service identity — settled

- **Classification:** Resolved clinical storage choice.
- **Evidence:** S21,following the user's service_id/join preference and rejection of hypothetical renaming as a snapshot requirement.
- **Answer:** Store mandatory service_id and display the name through a catalog join. Do not add a copied clinical treatment-name snapshot. Immutable invoice-line descriptions and amounts remain separate and required. Common recorder/server timestamp conventions remain settled.
- **Aliases:** The user mentions aliases as alternative wording on the catalog; exact field representation is schema work,not a reason to duplicate names on clinical occurrences.
- **Current disposition:** CLOSED MVP (R29,R82). Exact DDL remains within A03; this business choice is not reopened.

### Q07 — Existing catalog price/availability permissions — settled

- **Classification:** Resolved authorization rule.
- **Evidence:** S20,explicit user answer.
- **Answer:** Only admin may edit master catalog prices or enable/disable services. Reception cannot. Admin-only additions remain settled. Enforce the authority on the backend alongside UI restrictions.
- **Current disposition:** CLOSED MVP (R10,R80). Exact authorization implementation remains audited; do not reopen business authority.

### PC08 — Collection after interruption and on later days — settled

- **Evidence:** S24, owner reply on 29 October 2026.
- **Answer:** future reception may reopen and finish initial collection after interruption. Pay Later ends that initial interaction, while later positive receipts remain permitted on UNPAID/PARTIALLY_PAID invoices, including on another day. No admin handoff is required solely because collection is later. Only admin may waive.
- **Delivery phase:** MVP is admin-only. The separate reception role and corresponding authorization/RLS are post-MVP under S37 §9 and §22, superseding S24 phasing. This does not assert deployed role configuration.
- **Disposition:** CLOSED BUSINESS CLARIFICATION (R38–R40). Exact recovery, permissions/RLS, concurrency and state enforcement remain A07–A09. The old initial-checkout-only reception restriction is superseded.

## Parked and audit follow-up

These are recorded unfinished questions, not requests to restart the audit or package conversation now. AUDIT REQUIRED means technical work is unfinished; it does not assert that every individual missing column received a separate parking approval.

| ID | Workstream | Question to preserve | Original references | Register |
| --- | --- | --- | --- | --- |
| A01 | Independent examination/lapse fee events | How should independent patient-level extra_fees events, including first and recurrent fee-only checkout, be represented and constrained without a required complaint-specific visit_id, visit_services or clinical examination record? | S23; S11 Part I §8; S15 D-27; O02 | R15,R19,R69 |
| A02 | Reference ownership and API trust | Which patient/clinic/complaint identifiers should be stored versus derived, how is ownership constrained, and how should missing or contradictory client IDs be handled? | S16 M008; S15 §6; O13/O15 | R67 |
| A03 | Anatomical normalization and standalone schema | How should shared regions,complaint-catalog anatomy,mandatory standalone anatomy,service_id with joined names,and recorder/server timestamps be constrained without redundant course data? Q06 is settled; exact DDL remains open. | S15 D-28–D-32; Q06; O11 | R27,R28,R29,R68 |
| A04 | Attendance recency and fee integrity | How do therapy-only and fee-only completed attendance map to actual records, local dates and same-day behavior, first attendance, one charge per physical attendance, concurrent requests and historical/package visits? | S23; S11 Part I §§4–5; S15 §6; O03/O13/O14 | R15,R16,R17,R20 |
| A05 | Complaint lifecycle | Do complaints stay active indefinitely, how is explicit closure/inactivity handled, and what happens to an original complaint when a replacement is created? No closure policy is chosen. | S16 M144–M145,M171–M174; S15 D-40 | R08,R70 |
| A06 | Invoice sources and constraints | What are the exact invoice/line/extra-fee fields, source FKs, ownership/completeness checks, and once-per-occurrence uniqueness mechanisms? Count logical sources, not every contextual FK. | O07/O13; N28; S15 D-13,D-17; S17 U01–U02 | R19,R31,R32,R34,R35 |
| A07 | Settlement and authorization | Implement settled S24 phasing/authority: admin-only current phase; later reception may take initial/later receipts but never waive. Audit exact DDL, ownership, nonterminal metadata, state/amount validation, reception-specific RLS/roles and row-locking for finality/no overpayment. Admin-only catalog authority remains settled (Q07). | O09/O10/O15; N14–N17; S14 §6; S24 | R36–R56 |
| A08 | Idempotency and uncertain outcome | What request/key scope, canonical payload hash, retention, concurrency, replay and recovery are needed for checkout and payment after a lost response? Proposed key/hash/invoice storage is not final. | O14; N19; S15 D-24 | R60,R61 |
| A09 | Payment API, reporting and indexes | What payload/confirmation/cancel/error states, financial views/functions and payment indexes implement the settled behavior and recording-day reports without duplicate sums? | O14–O16; N16,N20,N31; S14 §8 | R41,R46,R54,R57,R59,R66,R72 |
| A10 | Migration and validation | How does final design differ from repository/deployed schema and legacy ledger states, what backfill/rollback is required, and which acceptance/concurrency checks must be executed? | O17/O18; N05,N22; S15 §6 | R66,R71,R72 |
| A11 | Dedicated package workstream | Reconciled through S37 (5 Nov 2026 decisions): Package business rules are settled. Shipping upgrades in MVP remains conditional on bounded engineering effort (S34 §6, A11/N51). Settled: admin-only MVP scope (reception post-MVP); ordinary 10- and 26-visit offerings in MVP; +16 visits for an eligible upgrade; fixed 30-consecutive-day review anchored to package start date ($T$ or $T+1$); daily visit cap default 1 (copied to upgrade); terminal parent eligibility for upgrade (Completed, or Closed on/after review date); upgrade pricing = current long package price minus original invoiced charge, floored at zero; upgrade review date keeps parent start-date anchor; invoice-level refund context displayed nonblockingly; one upgrade per parent with terminal finality; calculated-zero upgrade invoice followed by separate Mark as paid; payment-independent activation; strict closure irreversibility; immutable justification notes; six active closure choices including Other; "Duplicate Package Purchased" is deferred; the "Patient Stopped Attending" date gate remains; no advance purchase queue; single combined invoice; package_catalog and packages naming; questionnaire Q01–Q40 complete. Future scope: receptionist package debt block (receptionist cannot sell packages if patient has any unsettled package purchase invoices); ordinary care unblocked by debt. Technical redemption integrity DDL audits, concurrency/race condition guards, retry idempotency, server-controlled dates/prices, and controlled write paths remain open as engineering tasks (informed by S35–S36 scratch-model evidence). | O08; N26,N32,N51; S09 §6; Q01; S25–S37 | R33,R62,R74 |
| A12 | Historical schema dispositions | Audit dependency/data implications before dropping or retaining vitals, removing alerts, or changing timeline_events. Existing 16-table notes are historical, not a fresh deployment inventory. | N09–N11; S06 §5; S08 §4 | R65 |
| A13 | Parked frontend cleanup | What should happen to selected procedures when a complaint is deselected? The earlier cleanup/validation question remains explicitly parked. | N33; S09 §7 | Historical scope carry-forward |

Future-only items remain separate: retail/product-sale modeling; patient credit wallets, unallocated balances, inter-invoice payment reallocation, and generic invoice/line voiding (DR-01); multi-clinic recurrence/private catalog expansion; negotiated-rate engines; automated old-debt notifications and cross-invoice allocation. A planned refund module with mandatory reasons is in scope for MVP (S31 Q22). Their absence is intentional scope, not missing consolidation.

## Current issue dispositions

Current dispositions match the tracker. All seven review questions are closed. Historical source wording cannot override current decisions; engineering audits remain unfinished.

| Issue | Current status | Current interpretation |
| --- | --- | --- |
| N01 | DEFERRED FUTURE | Clinical voiding is excluded from MVP by S09 §§1–2, superseding open→optional scope. No unused void columns. |
| N02 | OPEN FUTURE | If voiding built, choose physical cascade versus derived parent validity and guard child inserts. S07 §2; S08 G13. |
| N03 | CONDITIONAL FUTURE | Admin-only voiding was selected if built. Enforcement/details remain open; feature deferred. S07 §2; S09 §2. |
| N04 | OPEN FUTURE | Void metadata constraints, reason, actor/time derivation and undo remain unselected. S08 G13; S09 §2. |
| N05 | AUDIT REQUIRED | Legacy ledger statuses and current financial queries need review; void filtering remains future-only. R66; A10. |
| N06 | OPEN FUTURE | Financial meaning of clinical void/filter and original billed vs collectible amounts remains open. No correction feature in MVP. S08 G06–G12; S09 §2. |
| N07 | DEFERRED FUTURE | Complaint-course retroactive correction/descendant semantics not approved for MVP. S09 §§1–2. |
| N08 | DEFERRED FUTURE | Clinical-note retroactive correction/deletion/void workflow not approved for MVP. S09 §§1–2. |
| N09 | AUDIT REQUIRED | Vitals feature deferred; drop versus dormant remains a dependency/data audit, not implemented cleanup. R65; A12. |
| N10 | AUDIT REQUIRED | Alerts removal intent is retained; inspect data/dependencies before any removal. R65; A12. |
| N11 | AUDIT REQUIRED | Investigate timeline_events purpose and usage before retention/removal. R65; A12. |
| N12 | DEFERRED FUTURE | No post-issuance append in MVP. Any future append/correction design requires separate authorization and review. R63–R64. |
| N13 | DEFERRED CORRECTION | Overcharge-correction workflows remain outside MVP. Ordinary final admin waiver is settled; only physical enforcement remains in A07. R40,R51–R56. |
| N14 | AUDIT REQUIRED | Invoice-level final waiver, derived amount, settled_at/settled_by and waiver-only reason are decided. Exact SQL/authorization remains audited. R51–R55; A07. |
| N15 | AUDIT REQUIRED | Positive paise, required recorder/server timestamps, immutability and deletion restriction are decided. DDL/security integration remains audited. R44–R50; A07. |
| N16 | AUDIT REQUIRED | Cash/UPI, explicit split inputs and UNPAID/PARTIALLY_PAID/PAID/WAIVED are settled. Exact API/form/error contract remains A09. S17 U02; R38,R42–R44. S21 terminology. |
| N17 | AUDIT REQUIRED | Admin now and the planned reception role may record repeated positive receipts on UNPAID/PARTIALLY_PAID invoices with collectible balance, including after Pay Later or on later days. Only admin may waive the full residual; WAIVED is terminal. No partial waiver remaining PARTIALLY_PAID. S24 supersedes the older initial-only reception restriction; exact SQL remains A07. R36,R39,R40,R55. |
| N18 | CLOSED MVP | created_at and received_at are non-null server timestamps. No client backdating; delayed entry belongs to the recording day. R46. |
| N19 | AUDIT REQUIRED | Idempotency requirement retained; key/hash/invoice mechanism provisional. Source uniqueness, invoice locking and replay are distinct controls. R35,R41,R60–R61; A08. |
| N20 | AUDIT REQUIRED | Derived money formulas and no extra stored monetary authorities are locked. Exact views/functions and safe aggregates remain audited. R54,R66; A09. |
| N21 | AUDIT REQUIRED | Standalone service_id with joined name,no clinical name snapshot,mandatory anatomy and recording conventions are settled. Exact physical DDL remains A03. S21; R25–R29,R82. |
| N22 | AUDIT REQUIRED | Compare actual schema with decided immutable lines,payments,settlement metadata and UNPAID/PARTIALLY_PAID/PAID/WAIVED states before migrations. No migration performed. R37,R53,R72; A10. S21 terminology. |
| N23 | OPEN FUTURE | Old visit_services creation metadata gap was identified conditionally; no new columns preapproved. S06 §5; S08 §4. |
| N24 | DEFERRED FUTURE | Paid financial corrections/refunds/credit and retroactive repair remain outside MVP. S09 §§1–2,7. |
| N25 | DEFERRED FUTURE | Automated old-debt alerts and receipt allocation across invoices remain excluded. S06 N25. |
| N26 | CLOSED MVP | Reconciled through S37: ordinary 10- and 26-visit offerings, admin-only MVP, payment-independent coverage from start date, review anchored to start date, permanent closure, six active closure choices including Other; "Duplicate Package Purchased" is deferred; the "Patient Stopped Attending" date gate remains, default daily cap 1, no advance purchase queue, and package_catalog/packages naming. Shipping upgrades in MVP remains conditional on bounded engineering effort (S34 §6, A11/N51). R62; A11; S25–S37. |
| N27 | OPEN FUTURE | Clinical correction source display and original-charge history if future voiding built. No current invoice amendment feature. S08 G10,G12; S09 §2. |
| N28 | AUDIT REQUIRED | Typed logical source default and once-per-occurrence uniqueness retained. Independent extra_fees needs no complaint-visit or performed-examination source; exact fee/source constraints remain audited. S23; R19,R34–R35; A01,A06. |
| N29 | OPEN FUTURE | Avoid waiver plus void-filter double financial reduction; define original vs eligible totals before corrections. S08 G06/G10. |
| N30 | OPEN FUTURE | Paid-linked false record trap, indirect invoice links, waiver-aware floor and zero-balance status after void remain unresolved. S08 G07–G11/G15. |
| N31 | AUDIT REQUIRED | Backend-confirmed success required; form preserved after confirmed rollback. Unknown outcome recovery and exact API/UI are audited. R59–R61; A08–A09. |
| N32 | CLOSED MVP | Package refund, transfer, and upgrade boundaries settled across S31, S33, S34, S37: admin-only refund module, transfers forbidden, narrow same-complaint 10→26 upgrade (+16 visits, current 26-visit catalog price minus the original 10-visit purchase-line invoiced charge, floored at zero, Day-30 window, terminal parent, start-date anchor, daily cap copy, one upgrade finality). Nonblocking invoice-level refund display. Shipping upgrades in MVP remains conditional on bounded engineering effort (S34 §6, A11/N51). R62; A11; S31; S33; S34; S37. |
| N33 | PARKED | Complaint deselection/procedure-cleanup validation remains explicitly parked. A13; S09 §7. |
| N34 | CLOSED MVP | Q01 resolved: permit package-covered-only zero invoice,validate coverage,at least one line; after issuance only Mark as paid,no payment row,zero collections. Positive uncovered fee uses normal settlement. S37 §25 adds the calculated-zero upgrade exception (R32, R36, R38). S17 U01; R74; S37 §25. |
| N35 | CLOSED MVP | Q02 settled: UNPAID,PARTIALLY_PAID,PAID,WAIVED are explicitly approved. The earlier PENDING/DUE labels are superseded; business behavior remains unchanged. S21; R37,R75,R81. |
| N36 | CLOSED CLARIFICATION | Q03/S23: examination and lapse fees are billing terms, not clinical examination records. Backend Create Invoice automatically includes applicable fees; no clinical examination prerequisite or receptionist confirmation gate. First and recurrent fee-only checkout follow the same rules. R18,R76. |
| N37 | CORRECTED | Q04/S23: performed-care records when present, applicable fees, invoice and lines commit atomically; fee-only checkout needs no dummy care. Payment follows separately after return. S12 Q3 exception was a source-summary error. S17 U04; R22,R23,R77. |
| N38 | CLOSED MVP | Q05 settled: clinical preview focuses on what was done today; no automatic fees or invoice prices/totals before Create Invoice. Show itemized fees and total on returned issued invoice. Existing last-visit context may remain. Fee calculation stays inside atomic issuance. S19; R21,R79. |
| N39 | CLOSED MVP | Q06 settled: mandatory service_id with catalog-name join; no extra clinical treatment-name snapshot. Invoice-line snapshots remain required; exact clinical DDL remains A03. S21; R29,R82. |
| N40 | CLOSED MVP | Q07 settled: only admin may edit existing master catalog prices or enable/disable services. Reception cannot. Admin-only additions remain settled; exact backend authorization enforcement remains audited. S20; R10,R80. |
| N41 | AUDIT REQUIRED | A01 — Independent examination/lapse fee events: How should independent patient-level extra_fees events, including first and recurrent fee-only checkout, be represented and constrained without a required complaint-specific visit_id, visit_services or clinical examination record? Source: S23; S11 Part I §8; S15 D-27; O02. |
| N42 | AUDIT REQUIRED | A02 — Reference ownership and API trust: Which patient/clinic/complaint identifiers should be stored versus derived, how is ownership constrained, and how should missing or contradictory client IDs be handled? Source: S16 M008; S15 §6; O13/O15. |
| N43 | AUDIT REQUIRED | A03: implement anatomical normalization and exact standalone service_id,ownership,recorder/server-timestamp constraints. Joined service name with no clinical name snapshot is settled; alias representation remains schema work. S21; R27–R29,R68,R82. |
| N44 | AUDIT REQUIRED | A04 — Attendance recency and fee integrity: How do therapy-only and fee-only completed attendance map to actual records, local dates and same-day behavior, first attendance, one charge per physical attendance, concurrent requests and historical/package visits? Source: S23; S11 Part I §§4–5; S15 §6; O03/O13/O14. |
| N45 | PARKED | A05 — Complaint lifecycle: Do complaints stay active indefinitely, how is explicit closure/inactivity handled, and what happens to an original complaint when a replacement is created? No closure policy is chosen. Source: S16 M144–M145,M171–M174; S15 D-40. |
| N46 | AUDIT REQUIRED | A06: exact invoice,line,extra-fee source FKs,ownership,completeness and once-per-occurrence constraints. Implement resolved zero-total exceptions (covered-only visits and calculated-zero package upgrades) and UNPAID, PARTIALLY_PAID, PAID and WAIVED status names. S17 U01–U02; S37 §25; R19,R31–R35. |
| N47 | AUDIT REQUIRED | A07/PC08 resolved by S24: future reception may resume initial collection and record later receipts, including after Pay Later; only admin may waive. Current phase is admin-only; reception-specific role/RLS is planned for post-MVP (S37 §9, §22). Exact DDL, ownership, metadata, state/amount validation, locking, no-overpayment and role enforcement remain audited. S20,S24; S37 §9, §22; R36–R56,R80. |
| N48 | AUDIT REQUIRED | A08 — Idempotency and uncertain outcome: What request/key scope, canonical payload hash, retention, concurrency, replay and recovery are needed for checkout and payment after a lost response? Proposed key/hash/invoice storage is not final. Source: O14; N19; S15 D-24. |
| N49 | AUDIT REQUIRED | A09 — Payment API, reporting and indexes: What payload/confirmation/cancel/error states, financial views/functions and payment indexes implement the settled behavior and recording-day reports without duplicate sums? Source: O14–O16; N16,N20,N31; S14 §8. |
| N50 | AUDIT REQUIRED | A10 — Migration and validation: How does final design differ from repository/deployed schema and legacy ledger states, what backfill/rollback is required, and which acceptance/concurrency checks must be executed? Source: O17/O18; N05,N22; S15 §6. |
| N51 | AUDIT REQUIRED | A11 package business decisions fully reconciled through S37. Technical redemption integrity DDL audits, concurrent last-visit consumption, retry idempotency, server-controlled validation, and race condition guards remain as engineering tasks. R62; A11; S25–S37. |
| N52 | AUDIT REQUIRED | A12 — Historical schema dispositions: Audit dependency/data implications before dropping or retaining vitals, removing alerts, or changing timeline_events. Existing 16-table notes are historical, not a fresh deployment inventory. Source: N09–N11; S06 §5; S08 §4. |
| N53 | PARKED | A13 — Parked frontend cleanup: What should happen to selected procedures when a complaint is deselected? The earlier cleanup/validation question remains explicitly parked. Source: N33; S09 §7. |

## Completion and validation boundary

This reconciliation accounts for incoming source units, the 44 MVP decision IDs, the 15 running-log decision IDs, all 176 exported messages, and inherited issue history. Repeated evidence is not counted as fresh business approval. Topic pages, workbook current columns and this ledger use the same fact/status register.

Source/archive hashes and retained dump files must match; existing history/research must remain unchanged. Current Markdown links and tracker source paths must resolve. The workbook must preserve its seven tabs, original C01–C54 historical columns, existing issue IDs and historical table audit, with updated current statuses, sources, bounded summary formulas and unexecuted acceptance cases.

Application code, database migrations, deployed schema, Git history and research conclusions are outside this documentation task. No acceptance scenario in the workbook is marked run by this consolidation. Unresolved Q/A entries prevent a false claim that complete Module 2 design is finished.
