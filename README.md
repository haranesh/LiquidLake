# LiquidLake

A comprehensive data lake platform that combines DuckDB, Apache Spark, Trino, and Nessie for modern data processing, versioning, and analytics.

## Overview

LiquidLake is a Docker-based data lake solution that integrates:

- **DuckDB**: Fast in-process SQL database for analytical queries
- **Apache Spark**: Distributed data processing framework
- **Trino**: Federated SQL query engine for querying multiple data sources
- **Nessie**: Git-like version control for data lake tables
- **Apache Iceberg**: Open-source table format for large-scale analytics
- **Jupyter**: Interactive notebooks for data exploration and analysis

## Architecture

```
┌─────────────────────────────────────────────────┐
│          Jupyter Notebooks                      │
│    (Interactive Data Analysis & Testing)        │
└──────────────┬──────────────────────────────────┘
               │
       ┌───────┴────────┬──────────┬──────────┐
       │                │          │          │
    ┌──▼──┐         ┌──▼──┐   ┌──▼──┐   ┌──▼──┐
    │Spark│         │Trino│   │Duck │   │HDFS │
    │     │         │     │   │DB   │   │     │
    └──────┘         └──┬──┘   └──┬──┘   └─────┘
                       │         │
                   ┌───▼─────────▼──┐
                   │   Nessie       │
                   │ (Versioning)   │
                   └────────────────┘
```

## Directory Structure

```
LiquidLake/
├── base/                 # Base Docker image with Hadoop ecosystem
│   ├── Dockerfile       # Multi-service base image
│   ├── entrypoint.sh    # Container initialization script
│   └── conf/            # Configuration files
│       ├── core-site.xml
│       ├── hdfs-site.xml
│       ├── hive-site.xml
│       ├── mapred-site.xml
│       ├── spark-defaults.conf
│       ├── yarn-site.xml
│       └── workers
├── jupyter/             # Jupyter service
│   ├── Dockerfile
│   ├── run.sh
│   └── notebook/        # Jupyter notebooks
├── trino/               # Trino configuration
│   ├── catalog/
│   │   └── iceberg.properties
│   ├── access-control.properties
│   └── rules.json
├── nessie_data/         # Nessie metadata storage
├── docker-compose.yml   # Service orchestration
├── Makefile             # Build and run commands
└── README.md           # This file
```

## Prerequisites

- Docker & Docker Compose
- Git
- Make (optional, for using Makefile commands)

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/haranesh/LiquidLake.git
cd LiquidLake
```

### 2. Build and Start Services

Using Docker Compose:

```bash
docker-compose up -d
```

Or using Make:

```bash
make build
make up
```

### 3. Access Services

- **Jupyter Notebooks**: http://localhost:8888
- **Trino CLI**: Access via `docker-compose exec trino trino`
- **Spark UI**: http://localhost:4040 (when a job is running)
- **HDFS NameNode**: http://localhost:9870

### 4. Run Sample Notebook

```bash
docker-compose exec jupyter jupyter notebook --ip=0.0.0.0 --allow-root
```

Navigate to `spark_connection_testing.ipynb` in the notebooks folder.

## Services

### DuckDB Container
Fast in-process SQL database optimized for analytical queries. Used for quick data exploration and testing.

### Apache Spark
Distributed computing framework for large-scale data processing. Configured with:
- YARN resource manager
- HDFS for distributed storage
- Hive metastore integration

### Trino
Federated SQL query engine that allows querying across multiple data sources including Spark, DuckDB, and Iceberg tables.

### Nessie
Provides Git-like version control for data lake tables. Enables:
- Branching and merging data
- Time travel queries
- Data lineage tracking

### Jupyter
Interactive notebooks for data exploration, analysis, and testing connections to all services.

## Configuration

### Spark Configuration
Edit `base/conf/spark-defaults.conf` to modify Spark settings.

### Trino Catalog
- **Iceberg Catalog**: Configured in `trino/catalog/iceberg.properties`
- Access control rules: `trino/access-control.properties`

### HDFS Configuration
- Core settings: `base/conf/core-site.xml`
- HDFS settings: `base/conf/hdfs-site.xml`

## Common Tasks

### Access Spark Shell
```bash
docker-compose exec spark spark-shell
```

### Query with Trino
```bash
docker-compose exec trino trino
```

### View HDFS
```bash
docker-compose exec hdfs hdfs dfs -ls /
```

### Check Service Logs
```bash
docker-compose logs -f <service-name>
```

## Troubleshooting

### DuckDB Segmentation Fault
If you encounter a segmentation fault from the DuckDB service:

1. **Check system resources**: Ensure sufficient memory and CPU are available
2. **Update DuckDB**: Update to the latest stable version in the Dockerfile
3. **Review logs**: 
   ```bash
   docker-compose logs LiquidLake_duckdb
   ```
4. **Restart service**:
   ```bash
   docker-compose restart LiquidLake_duckdb
   ```

### Spark Connection Issues
- Verify YARN is running: `docker-compose exec spark yarn application -list`
- Check HDFS status: `docker-compose exec hdfs hdfs dfsadmin -report`

### Trino Query Failures
- Verify catalog configuration: `docker-compose logs trino`
- Test Iceberg catalog connectivity through the Trino CLI

## Development

### Adding New Notebooks
Place Jupyter notebooks in `jupyter/notebook/` directory. They'll be available when Jupyter starts.

### Modifying Service Configuration
1. Edit the appropriate config file in `base/conf/`
2. Rebuild the base image: `docker-compose build base`
3. Restart services: `docker-compose up -d`

### Building Custom Images
Edit the Dockerfiles in respective service directories and rebuild:

```bash
docker-compose build <service-name>
docker-compose up -d <service-name>
```

## Environment Variables

Key environment variables can be configured in `docker-compose.yml`:

- `SPARK_MASTER`: Spark master URL
- `HADOOP_HOME`: Hadoop installation directory
- `JAVA_HOME`: Java installation directory

## Performance Tuning

### Spark
Adjust in `base/conf/spark-defaults.conf`:
```
spark.executor.memory
spark.driver.memory
spark.executor.cores
```

### YARN
Modify `base/conf/yarn-site.xml` for resource allocation

## Data Lake Best Practices

1. **Use Iceberg** for production tables with ACID guarantees
2. **Enable Nessie** for data versioning and governance
3. **Partition tables** by date or appropriate dimensions
4. **Use Trino** for cross-source queries
5. **Monitor** resource usage in YARN UI

## Contributing

For bug reports and feature requests, please open an issue in the repository.

## License

This project is free to use under the MIT License. Feel free to use, modify, and distribute this code as you see fit.

## Support

For support and questions, please refer to:
- [Nessie Documentation](https://projectnessie.org/)
- [Iceberg Documentation](https://iceberg.apache.org/)
- [DuckDB Documentation](https://duckdb.org/)
- [Apache Spark Documentation](https://spark.apache.org/)
- [Trino Documentation](https://trino.io/)

## Contact

If you need any help or would like to discuss this project, feel free to reach out:
- [Connect on LinkedIn](https://www.linkedin.com/in/hsokharaya/)

## Acknowledgments

Built on top of excellent open-source projects:
- Apache Spark
- Apache Hadoop
- DuckDB
- Trino
- Project Nessie
- Apache Iceberg
