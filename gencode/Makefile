# pwd named "release_#"
GC_VER 		?= $(word 2,$(subst _, ,$(notdir $(PWD))))
GTF 		 = gencode.v$(GC_VER).annotation.gtf
SUMMARIES 	 = gene_id2name.txt isoform2gene.txt
TBX_GTF		 = $(GTF).bgz $(GTF).bgz.tbi
LOAD_TABIX 	?= module load ngs-ccts/tabix

default: $(GTF) $(SUMMARIES) $(TBX_GTF)

pull: $(GTF).gz 

# DNS doesn't resolve on our cluster 
ftp.sanger.ac.uk ?= 193.62.203.17
$(GTF).gz: 
	wget "ftp://$(ftp.sanger.ac.uk)/pub/gencode/Gencode_human/release_$(GC_VER)/$(GTF).gz"
	md5sum $@ > $@.md5

$(GTF).bgz: $(GTF).gz
	$(LOAD_TABIX) \
	; zcat $< \
	|sort -k1,1 -k4,4n -k5,5n \
	| bgzip -c \
	> $@

$(GTF).bgz.tbi: $(GTF).bgz
	$(LOAD_TABIX) \
	; tabix -p gff $<	

%: %.gz
	zcat $< > $@

isoform2gene.txt: $(GTF)
	awk 'BEGIN{OFS="\t";header=0;print "#gene_id","gene_name","transcript_id";}($$3=="gene"||$$3=="transcript"){gene_id="";gene_name="";transcript_id="";for(i=9; i<NF;i=i+2){if($$i=="gene_id"){split($$(i+1),a,"[\".]");gene_id=a[2];} else if($$i=="gene_name"){split($$(i+1),b,"[\"]");gene_name=b[2];} else if($$i=="transcript_id"){split($$(i+1),c,"[\".]");transcript_id=c[2];}};print gene_id,gene_name,transcript_id}' $< \
	| sort -k1,1 -k3,3 \
	> $@

gene_id2name.txt: isoform2gene.txt
	cut -f 1,2 $< \
	| sort | uniq \
	> $@



# extract records
# same result as : grep -w ATXN7L1 gencode.v19.annotation.gtf
extract: $(GTF) \
	ATXN7L1.gene_txn.gtf ATXN7L1_out.gtf  \
	TNFRSF4.gene_txn.gtf TNFRSF4_out.gtf 

%_out.gtf : $(GTF) ensembl_acc_%.txt
	(for x in $$(cat ensembl_acc_$*.txt ); do grep $$x $< ; done) | sort | uniq > $@

%.gene_txn.gtf: $(GTF)
	grep -w $* $< | awk '($$3=="gene"||$$3=="transcript"){print $$0}' > $@
	wc -l $@
