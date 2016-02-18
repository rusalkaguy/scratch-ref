GTF	?= Master_hg19_RNA.gtf

SUMMARIES = gene_id2name.txt isoform2gene.txt
# update gene names
MAPPED_FILE	?= isoform2gene_v24mapped.txt
GENCODE_DIR	?= ../../gencode/Gencode_human/release_24
GC_MAPFILE	?= gencode.v24.gene_id2name.txt
REFSEQ_MAPFILE 	?= ucsc.hg19.refseq.id2name.txt
UCSC_MAPFILE	?= ucsc.hg19.knowngene.id2name.txt
UNIFIED_MAPFILE	?= gc.ucsc.refseq.id2name.txt

default: $(SUMMARIES) $(MAPPED_FILE) $(UNIFIED_MAPFILE)

isoform2gene.txt: $(GTF)
	echo -e "#gene_id\tgene_name\ttranscript_id" > $@
	awk 'BEGIN{OFS="\t";header=0}(($$3=="gene"||$$3=="transcript")&&($$9=="gene_id"&&$$17=="gene_name"&&$$11=="transcript_id")){if(!header){print "#" $$9,$$17,$$11;header=1};split($$10,a,"[\".]");gene_id=a[2];split($$18,a,"[\"]");gene_name=a[2];split($$12,a,"[\".]");transcript_id=a[2];print gene_id,gene_name,transcript_id}' $< \
	| sort -k1,1 -k3,3 \
	>> $@

gene_id2name.txt: isoform2gene.txt
	grep "^#" $< | cut -f 1,2 > $@
	grep -v "^#" $<	| cut -f 1,2 | sort | uniq >> $@

#
# apply gene name map
#
map: $(MAPPED_FILE)
$(MAPPED_FILE): isoform2gene.txt $(GC_MAPFILE)
	echo "# source $(PWD)/$<" > $@
	echo "# gene_names updated from $(GC_MAPFILE)" >> $@
	join -a 1 -t "	" $< $(GC_MAPFILE) \
	| awk 'BEGIN{OFS="\t"}{if($$4!=""){$$2=$$4};print $$1,$$2,$$3}' \
	>> $@

# 
# external id/gene/txn maps
#
# Gencode
$(GENCODE_DIR)/gene_id2name.txt:
	cd $(dir $@); make $(notdir $@)
$(GC_MAPFILE): $(GENCODE_DIR)/gene_id2name.txt
	ln -s $< $@

# UCSC
../../ucsc/hg19/knowngene/id2name.txt:
	cd $(dir $@); make $(notdir $@)
$(UCSC_MAPFILE): ../../ucsc/hg19/knowngene/id2name.txt
	ln -s $< $@
# RefSeq
../../ucsc/hg19/refseq/id2name.txt:
	cd $(dir $@); make $(notdir $@)
$(REFSEQ_MAPFILE): 
	ln -s $< $@

# combine all 3 into one file
$(UNIFIED_MAPFILE): $(MAPPED_FILE) $(UCSC_MAPFILE) $(REFSEQ_MAPFILE)
	awk '(FILENAME != prevFN){print "#" FILENAME; prevFN=FILENAME}{print $$0}' $^ > $@

# 
# utilities
#
%.gtf: %.gtf.gz
	gunzip -c $< > $@

clean:
	rm $(GTF) $(SUMMARIES) $(MAPPED_FILE) 
