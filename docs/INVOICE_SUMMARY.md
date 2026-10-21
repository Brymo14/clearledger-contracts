# Invoice summary reads

List views should use the invoice's compact summary fields: identifier,
participants, face value, due date, status, and funding asset. Detailed
attestation and metadata payloads are separate concerns and should only be
loaded for an invoice detail view, keeping pagination responses bounded.
