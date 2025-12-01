# Pressure Simulation

🔥 1. CPU Load Generator (Adjustable Load % or Core Count)  
Script: `cpu-load-variable.sh`
```
#!/bin/bash

##########################################
# Adjustable CPU Load Generator (RHEL 8)
# Press Ctrl+C to stop
##########################################

CPU_CORES=2       # Number of CPU cores to stress
LOAD_PERCENT=80   # Target load per core (best-effort)

echo "🔥 Starting CPU load:"
echo "  Cores: $CPU_CORES"
echo "  Load : $LOAD_PERCENT% per core"
echo "Press Ctrl+C to stop."
echo

for i in $(seq 1 $CPU_CORES); do
{
    while true; do
        # Busy loop (LOAD_PERCENT %)
        busy=$(( LOAD_PERCENT * 1000 / 100 ))
        idle=$(( 1000 - busy ))

        # Busy phase
        start=$(date +%s%3N)
        while [ $(( $(date +%s%3N) - start )) -lt $busy ]; do :; done

        # Idle phase
        sleep $(awk "BEGIN { print $idle/1000 }")
    done
} &
done

wait
```

🧠 Memory Pressure + Swap Load Script  
Script: `memory-pressure-swap.sh`
```
#!/bin/bash

##########################################################
# Memory Pressure + Swap Usage Simulator (RHEL 8)
# Creates real memory pressure and forces swapping.
# Press Ctrl+C to stop.
##########################################################

MEMORY_MB=6000      # How much RAM to allocate (MB)
CHUNK_MB=200        # Allocate in chunks (MB) - safer

echo "🧠 Starting Memory Pressure Tool"
echo "  Total target allocation : $MEMORY_MB MB"
echo "  Chunk allocation size    : $CHUNK_MB MB"
echo "  Will generate swap usage if RAM is exceeded."
echo "Press Ctrl+C to stop."
echo ""

# Convert MB → bytes
CHUNK_BYTES=$(( CHUNK_MB * 1024 * 1024 ))
TOTAL_BYTES=$(( MEMORY_MB * 1024 * 1024 ))
ALLOCATED=0

declare -a blocks

# Allocate memory gradually to avoid OOM-kill
while [ $ALLOCATED -lt $TOTAL_BYTES ]; do
    echo "🟡 Allocating ${CHUNK_MB}MB (Total so far: $((ALLOCATED/1024/1024)) MB)"
    
    # Allocate chunk
    block=$(dd if=/dev/zero bs=$CHUNK_BYTES count=1 2>/dev/null | tr '\0' 'A')
    blocks+=("$block")
    
    ALLOCATED=$((ALLOCATED + CHUNK_BYTES))
    sleep 0.5
done

echo "🟢 Allocation complete: $((ALLOCATED/1024/1024)) MB allocated."
echo "🔄 Keeping memory hot to force swap activity..."

# Keep memory "dirty" so Linux cannot free it
while true; do
    for i in "${!blocks[@]}"; do
        blocks[$i]=$(echo "${blocks[$i]}" | sed 's/A/B/g')
    done
    sleep 1
done
```
✅ Network Load Script With Adjustable Mbps  
Script: `net-load-variable.sh`
```
#!/bin/bash

##########################################
# Adjustable Mbps Network Load Generator
# Works on RHEL 8
# Press Ctrl+C to stop
##########################################

RX_Mbps=50     # Download load
TX_Mbps=30     # Upload load

DOWNLOAD_URL="http://speedtest.tele2.net/100MB.zip"
UPLOAD_URL="https://httpbin.org/post"

# Convert Mbps → bytes per second (Mbps * 1024*1024 / 8)
RX_BYTES=$(( RX_Mbps * 1024 * 1024 / 8 ))
TX_BYTES=$(( TX_Mbps * 1024 * 1024 / 8 ))

echo "🚀 Network load generator started"
echo "⬇️ Download Load (RX): ${RX_Mbps} Mbps"
echo "⬆️ Upload Load   (TX): ${TX_Mbps} Mbps"
echo "Press Ctrl+C to stop."

while true; do
    # RX Load Generator
    (
        curl -s "$DOWNLOAD_URL" --limit-rate "$RX_BYTES" > /dev/null
    ) &

    # TX Load Generator (upload from /dev/zero)
    (
        dd if=/dev/zero bs="$TX_BYTES" count=1 2>/dev/null | \
        curl -s -X POST -F "file=@-" "$UPLOAD_URL" > /dev/null
    ) &

    sleep 1
done
```
🔥 Disk Read + Write Load Generator (Adjustable MB/s)  
Script: `disk-load-variable.sh`
```
#!/bin/bash

##########################################################
# Adjustable Disk Read/Write Load Generator (RHEL 8)
# Generates controlled MB/s disk IO load.
# Press Ctrl+C to stop.
##########################################################

WRITE_MBPS=50     # Target write throughput (MB/s)
READ_MBPS=80      # Target read throughput (MB/s)
TESTFILE="/tmp/diskload.test"  # Temporary test file

echo "🚀 Disk Load Generator Started"
echo "  Write Load : ${WRITE_MBPS} MB/s"
echo "  Read  Load : ${READ_MBPS} MB/s"
echo "  Test file  : $TESTFILE"
echo "Press Ctrl+C to stop."
echo

# Convert MBps → bytes/sec
WRITE_BPS=$(( WRITE_MBPS * 1024 * 1024 ))
READ_BPS=$(( READ_MBPS * 1024 * 1024 ))

# Make 1GB test file if not exists
if [ ! -f "$TESTFILE" ]; then
    echo "🟡 Creating test file (1GB)..."
    dd if=/dev/zero of="$TESTFILE" bs=1M count=1024 status=none
    echo "🟢 Test file created."
fi

########################################
# Write Load Function
########################################
write_load() {
    while true; do
        dd if=/dev/zero of="$TESTFILE" bs="$WRITE_BPS" count=1 conv=notrunc oflag=direct 2>/dev/null
        sleep 1
    done
}

########################################
# Read Load Function
########################################
read_load() {
    while true; do
        dd if="$TESTFILE" of=/dev/null bs="$READ_BPS" count=1 iflag=direct 2>/dev/null
        sleep 1
    done
}

# Start both loads in background
write_load &
WRITE_PID=$!

read_load &
READ_PID=$!

# Wait forever until user stops
wait
```
