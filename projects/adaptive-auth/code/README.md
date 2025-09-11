# co.rishith.adaptive.main.flow  (minimal adaptive auth)

# 1) Ask for username/password
creds = RRF "login.ftlh"

# 2) Check password (demo creds)
WHEN (creds.username is "user") AND (creds.password is "pass") THEN
    # 3) Compute risk by hour (off-hours 22:00–05:59 → step-up)
    now  = CALL STATIC java.time.LocalTime now
    hour = CALL METHOD now getHour

    WHEN (hour is 0) OR (hour is 1) OR (hour is 2) OR (hour is 3) OR
         (hour is 4) OR (hour is 5) OR (hour is 22) OR (hour is 23) THEN
        # 4) Step-up to OTP
        otp = RRF "otp.ftlh"
        WHEN (otp.code is "123456") THEN
            FINISH SUCCESS { userid: creds.username }
        ELSE
            FINISH ERROR { "error": "Invalid OTP" }
        END
    ELSE
        # 5) Low risk → password only
        FINISH SUCCESS { userid: creds.username }
    END
ELSE
    FINISH ERROR { "error": "Invalid username or password" }
END
